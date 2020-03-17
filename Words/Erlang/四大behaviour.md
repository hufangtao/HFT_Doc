# erlang四大behaviour总结

[参考](https://blog.csdn.net/chenyefei/article/details/70195637 "")

## supervisor
#### 1. 什么是supervisor
监督者负责启动、停止、监控他的子进程。
监督者负责重启子进程，保证子进程的存活性。

#### 2. 实例
监督者的回调模块
``` erlang
init(...) ->   
 {ok, {{RestartStrategy, MaxR, MaxT},       
   [ChildSpec, ...]}}.
```
- RestartStrategy: 重启策略
- MaxR，MaxT: 最大启动频率
- [ChildSpec, ...]：子进程列表
###### 例：
``` erlang
-module(ch_sup).
-behaviour(supervisor).
-export([start_link/0]).
-export([init/1]).
start_link() ->    
    supervisor:start_link(ch_sup, []).
init(_Args) ->   
    {ok, {{one_for_one, 1, 60},      
    [{ch3, {ch3, start_link, []},
    permanent, brutal_kill, worker, [ch3]}]}}.
```
one_for_one是重启策略
1和60定义了最大重启频率
{ch3, …}是子规程

#### 3. 重启策略
simple_one_for_one 
one_for_one 进程终止，仅仅这个进程会被重启
one_for_all 进程终止，终止其他子进程，然后重启所有子进程
rest_for_one 进程终止，

#### 4. 最大启动频率
**这是为了预防一个进程因某种原因频繁重启**
```
在MaxT时间内重启次数大于MaxR，监督者进程就停止它的所有进程，然后再终止自己。
```
#### 5. 子进程
ChildSpec定义如下：
``` erlang
ChildSpec = {Id, StartFunc, Restart, Shutdown, Type, Modules}   
Id = term()   
StartFunc = {M, F, A}  
M = F = atom()   
A = [term()] 
Restart = permanent | transient | temporary  
Shutdown = brutal_kill | integer() &gt;=0 | infinity   
Type = worker | supervisor    
Modules = [Module] | dynamic      
Module = atom()
```
- Id：子进程内部标识
- StartFunc: 一般用于调用gen_server的start_link
- Restart：进程终止后，如何重启
    - permanent 总是重启
    - temporary 从不重启
    - transient 不正常终止后才重启
- Shutdown：定义进程将如何被终止
    - krutal_kill 进程被exit无条件终止
- Type：指定子进程是sup还是worker
- Modules：是有一个元素的列表，假如子进程是gen_event那么Modules应是dynamic，其他的是回调模块的名称。

#### 6. 启动supervisor
``` erlang
start_link() ->
    supervisor:start_link(sup, []).
```
回调监督者进程调用init
``` erlang
init(_Args) ->
    {ok, {{one_for_one, 1, 60},
    [{sup, {sup, start_link, []},
    permanent, brutal_kill, worker, [sup]}]}}.
```
supervisor:start_link是同步的，它一直等到所有子进程都启动了才返回

#### 7. 添加子进程
``` erlang
supervisor:start_child(Sup, ChildSpec)
```
向监督者动态添加子进程
**监督者死掉重启，所以动态添加的子进程都不复存在**

#### 8. 通知子进程
``` erlang
supervisor:terminate_child(Sup, Id)
```

#### 9. simple_one_for_one重启策略
是one_for_one的简版，所有子进程都是同一进程实例而被动态添加。
实例：
``` erlang
-module(simple_sup).
-behaviour(supervisor).
-export([start_link/0]).
-export([init/1]).
start_link() ->    supervisor:start_link(simple_sup, []).
init(_Args) -> 
   {ok,
    {
       {simple_one_for_one, 0, 1},
       [{call, {call, start_link, []}, 
         temporary, brutal_kill, worker, 
         [call]
       }]
    }
   }.
```
当启动时，监督者不启动任何子进程，取而代之的是所有子进程都通过调用supervisor:start_child(Sup, List)来动态添加，Sup 是监督者的pid或名称，List 是添加给子规范中指定参数列表term列表，如果启动函数是{M, F, A}这种形式，那么子进程通过调用apply(M, F, A++List)而被启动

``` erlang
supervisor:start_child(Pid, [id1])
```
那么子进程通过调用apply(call, start_link, [] ++ [id1])而被启动，实际上就是call:start_link(id1)

---
## gen_server
什么是gen_server
#### 源码解析
gen_server模块中首先定义了behaviour的回调函数，所有behaviour为gen_server的模块都要定义的回调。

----
初始化Mod，负责gen_server的初始化，主要初始化State。
``` erlang
-callback init(Args :: term()) ->
    {ok, State :: term()} | {ok, State :: term(), timeout() | hibernate | {continue, term()}} |
    {stop, Reason :: term()} | ignore.
```
消息处理，负责发送给Mod的消息，修改State，返回修改后的State。
``` erlang
-callback handle_call(Request :: term(), From :: {pid(), Tag :: term()},
                      State :: term()) ->
    {reply, Reply :: term(), NewState :: term()} |
    {reply, Reply :: term(), NewState :: term(), timeout() | hibernate | {continue, term()}} |
    {noreply, NewState :: term()} |
    {noreply, NewState :: term(), timeout() | hibernate | {continue, term()}} |
    {stop, Reason :: term(), Reply :: term(), NewState :: term()} |
    {stop, Reason :: term(), NewState :: term()}.
-callback handle_cast(Request :: term(), State :: term()) ->
    {noreply, NewState :: term()} |
    {noreply, NewState :: term(), timeout() | hibernate | {continue, term()}} |
    {stop, Reason :: term(), NewState :: term()}.
-callback handle_info(Info :: timeout | term(), State :: term()) ->
    {noreply, NewState :: term()} |
    {noreply, NewState :: term(), timeout() | hibernate | {continue, term()}} |
    {stop, Reason :: term(), NewState :: term()}.
-callback handle_continue(Info :: term(), State :: term()) ->
    {noreply, NewState :: term()} |
    {noreply, NewState :: term(), timeout() | hibernate | {continue, term()}} |
    {stop, Reason :: term(), NewState :: term()}.
-callback terminate(Reason :: (normal | shutdown | {shutdown, term()} |
                               term()),
                    State :: term()) ->
    term().
-callback code_change(OldVsn :: (term() | {down, term()}), State :: term(),
                      Extra :: term()) ->
    {ok, NewState :: term()} | {error, Reason :: term()}.
-callback format_status(Opt, StatusData) -> Status when
      Opt :: 'normal' | 'terminate',
      StatusData :: [PDict | State],
      PDict :: [{Key :: term(), Value :: term()}],
      State :: term(),
      Status :: term().

-optional_callbacks(
    [handle_info/2, handle_continue/2, terminate/2, code_change/3, format_status/2]).
```
除了定义behaviour接口之外，还有一些接口
- start ：启动一个server
- start_link ： 启动一个server，链接到进程树
- stop ：停止server
- call ： 处理同步消息
- cast ： 处理异步消息
- 其它内部函数
    - 源码中gen_server的init_it函数用于回调Mod的初始化init(Args)函数。**init(Args)实现中初始化State。**
    - 在gen_server启动之后，进入一个loop，loop循环接受消息。当接收到消息后交给Mod:handle_call来处理。**所以强制要求所有gen_server的Mod都要实现handle_call。**
        ``` erlang
        receive
            Msg ->
	        decode_msg(Msg, Parent, Name, State, Mod, infinity, HibernateAfterTimeout, Debug, false)
        after HibernateAfterTimeout ->
	        loop(Parent, Name, State, Mod, hibernate,HibernateAfterTimeout, Debug)
        end;
        ```

#### gen_server的工作流:
User module||Generic
--|:--:|--:
start|----->|start
init|<-----|.
|||loop
handle_call|<-----|.
||----->|reply
handle_cast|<-----|.
handle_info|<-----|.
terminate|<-----|.
||----->|reply

---
## gen_fsm 

---
## gen_event 