# erlang四大behaviour总结

[参考](https://blog.csdn.net/chenyefei/article/details/70195637 "")

## supervisor

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


## gen_fsm 

## gen_event 