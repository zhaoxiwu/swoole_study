#Tcp Clinet

demo:

    $client = new swoole_client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_ASYNC);
    //设置事件回调函数
    $client->on("connect", function($cli) {
        $cli->send("hello world\n");
    });
    $client->on("receive", function($cli, $data){
        echo "Received: ".$data."\n";
    });
    $client->on("error", function($cli){
        echo "Connect failed\n";
    });
    $client->on("close", function($cli){
        echo "Connection close\n";
    });
    //发起网络连接
    $client->connect('127.0.0.1', 9501, 0.5);


#swoole_client

    PHP_METHOD(swoole_client, __construct){
        zend_parse_parameters(ZEND_NUM_ARGS(),"l|ls", type, async, id, len)
        //如果SwooleG.main_reactor不存在则创建，并初始化swoole_event,设置读写事件对应handler
        php_swoole_check_reactor(){
            if SwooleG.main_reactor = NULL {
                SwooleG.main_reactor = sw_malloc()
                swReactor_create(SwooleG.main_reactor, SW_REACTOR_MAXEVENTS){
                    //reactor event create, use epoll/kqueue/poll，已epoll为例
                    swReactorEpoll_create(reactor, max_envent)
                    reactor->running = 1

                    //set Handle, and onXXX methods, reactor->defer/write/close
                    reactor->setHandle = swReactor_setHandle;
                    reactor->onFinish = swReactor_onFinish;
                    reactor->onTimeout = swReactor_onTimeout;
                    reactor->write = swReactor_write;
                    reactor->defer = swReactor_defer;
                    reactor->close = swReactor_close;

                    reactor->socket_array = swArray_new(1024)
                }
            }

            //event init
            php_swoole_event_init(){
                //set read/write/error event and callbacks
                wooleG.main_reactor->setHandle(SwooleG.main_reactor, SW_FD_USER | SW_EVENT_READ, php_swoole_event_onRead);
                wooleG.main_reactor->setHandle(SwooleG.main_reactor, SW_FD_USER | SW_EVENT_WRITE, php_swoole_event_onWrite);
                wooleG.main_reactor->setHandle(SwooleG.main_reactor, SW_FD_USER | SW_EVENT_ERROR, php_swoole_event_onError);
            }
        }
    }

#swReactor_setHandle

    src/reactor/ReactorBase.c:79
    int swReactor_setHandle(swReactor *reactor, int _fdtype, swReactor_handle handle){
        //check fdtype rangs

        //add to handler,根据fdtype类型不同，放到不同的hanle里面
        //handle, write_handle, error_handle
        reactor->handle[fdtype] =handle;
    }

