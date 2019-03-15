# erlang自定义behaviour使用总结

### why自定义behaviour
behaviour相当于其他语言中的接口。假如我们有test_behaviour，所有实现test_behaviour的Mod都要实现test_behaviour中定义的回调函数。之所以叫回调函数，当某个Msg到达的时候，将会调用Mod::callback函数来实现某些功能。

### how自定义behaviour
以gen_server的实现为例，这是gen_server的init规定
``` erlang
-callback init(Args :: term()) ->
    {ok, State :: term()} | {ok, State :: term(), timeout() | hibernate | {continue, term()}} |
    {stop, Reason :: term()} | ignore.
```
在Mod中的实现：
``` erlang
init({Id, {ClientPid, IPStr, LoginKey, Platform, Partner, PlatformParams}} = Args) ->
  {ok, Player}.
```