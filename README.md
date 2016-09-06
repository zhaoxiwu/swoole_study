# swoole_study
代码分析路线按官方文档demo脉络进行
http_server, tcp_server, websocket_server, tcp_client, async_io, task


全局变量：

swooleG: 全局变量，进程之间不共享，可写

swooleGS：全局变量，进程之间共享，不可写

swooleWG:  Worker级别的全局变量

    enum php_swoole_server_callback_type
    {
        //--------------------------Swoole\Server--------------------------
        SW_SERVER_CB_onConnect,        //worker(event)
        SW_SERVER_CB_onReceive,        //worker(event)
        SW_SERVER_CB_onClose,          //worker(event)
        SW_SERVER_CB_onPacket,         //worker(event)
        SW_SERVER_CB_onStart,          //master
        SW_SERVER_CB_onShutdown,       //master
        SW_SERVER_CB_onWorkerStart,    //worker(event & task)
        SW_SERVER_CB_onWorkerStop,     //worker(event & task)
        SW_SERVER_CB_onTask,           //worker(task)
        SW_SERVER_CB_onFinish,         //worker(event & task)
        SW_SERVER_CB_onWorkerError,    //manager
        SW_SERVER_CB_onManagerStart,   //manager
        SW_SERVER_CB_onManagerStop,    //manager
        SW_SERVER_CB_onPipeMessage,    //worker(evnet & task)
        //--------------------------Swoole\Http\Server----------------------
        SW_SERVER_CB_onRequest,        //http server
        //--------------------------Swoole\WebSocket\Server-----------------
        SW_SERVER_CB_onHandShake,      //worker(event)
        SW_SERVER_CB_onOpen,           //worker(event)
        SW_SERVER_CB_onMessage,        //worker(event)
        //-------------------------------END--------------------------------
    };

    #define PHP_SERVER_CALLBACK_NUM             (SW_SERVER_CB_onMessage+1)

MINIT:

    {
        初始化过程, 顾名思义都是做一些基本的对象初始化，内存分配的。服务启动操作都在用户php层面操作
        swoole_init();
        swoole_server_port_init(module_number TSRMLS_CC);
        swoole_client_init(module_number TSRMLS_CC);
        #ifdef SW_COROUTINE
        swoole_client_coro_init(module_number TSRMLS_CC);
        #ifdef SW_USE_REDIS
        swoole_redis_coro_init(module_number TSRMLS_CC);
        #endif
        swoole_mysql_coro_init(module_number TSRMLS_CC);
        swoole_http_client_coro_init(module_number TSRMLS_CC);
        swoole_coroutine_util_init(module_number TSRMLS_CC);
        #endif
        swoole_http_client_init(module_number TSRMLS_CC);
        swoole_async_init(module_number TSRMLS_CC);
        swoole_process_init(module_number TSRMLS_CC);
        swoole_table_init(module_number TSRMLS_CC);
        swoole_lock_init(module_number TSRMLS_CC);
        swoole_atomic_init(module_number TSRMLS_CC);
        swoole_http_server_init(module_number TSRMLS_CC);
        swoole_buffer_init(module_number TSRMLS_CC);
        swoole_websocket_init(module_number TSRMLS_CC);
        swoole_mysql_init(module_number TSRMLS_CC);
        swoole_module_init(module_number TSRMLS_CC);
    };

