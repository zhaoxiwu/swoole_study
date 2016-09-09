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

#swoole_client->on

    PHP_METHOD(swoole_client, on){
        //大体逻辑跟其他几个server类似
        zend_parse_parameters(ZEND_NUM_ARGS, "sz", &cb_name, $cb_name_len, &zcallback)

        //获取clent的async type,
        zend_read_property(swoole_client_class_entry_ptr, getThis(), "type"，0)

        //给client的callback属性分配内存
        client_callback *cb = swoole_get_property(getThis,client_property_callback)
        cb = emalloc()
        swoole_set_property(getThis, client_property_callback, cb)

        //检查callback的可执行性，然后获取function name
        zend_is_callable(zcallback,0,func_name)

        //set connect/receive/close/error callback, adnd only support this 4 callback
        zend_update_property(swoole_client_class_entry_ptr, getThis, "onXXX", zcallback)
        cb->onXXX = zend_read_property(swoole_client_class_entry_ptr, getThis,"onXXX")
    }

#swoole_client->connect

    PHP_METHOD(swoole_client, connect){
        zend_parse_parameters(ZEND_NUM_ARGS, "s|ldl", &host, &host_len, &port, &timeout, &sock_flag)

        swClient *cli = swoole_get_object(getThis)
        cli = php_swoole_client_new(getThis, host, host_len, port){
            
        }
    }

