HttpServer demo

    $serv = new swoole_http_server("127.0.0.1", 9502);

    $serv->on('Request', function($request, $response) {
        var_dump($request->get);
        var_dump($request->post);
        var_dump($request->cookie);
        var_dump($request->files);
        var_dump($request->header);
        var_dump($request->server);

        $response->cookie("User", "Swoole");
        $response->header("X-Server", "Swoole");
        $response->end("<h1>Hello Swoole!</h1>");
    });

    $serv->start();

swoole_http_server(host, port)

    //其实看代码，没有显示定义swoole_http_server的构造函数，它继承了swoole_server类，该类构造
    //函数可以通过指定sock_type,支持不同协议类型的服务
    //在MINIT中会调用swoole_http_server_init(module_number), 该接口实现如下：
    swoole_http_server_init(module_number){
        init_class_entry(swoole_http_server_ce, "swoole_http_server", swoole_http_server_methods)
        //该函数的意思是，定义一个php框架的类：swoole_http_server, 父类为：swoole_server
        zend_register_internal_class_ex(&swoole_http_server_ce, swoole_server_class_entry_ptr， "swoole_server")
        //定义一个类别名"swoole\\Http\server"
        zend_register_class_alias("swoole\\http\\server", swoole_server_class_entry_ptr)

        //swoole_http_server增加一个属性，global
        zend_declare_property_long(swoole_server_class_entry_ptr, "global",0, ZEND_ACC_PRIVATE)

        //定义一个swoole_http_request, swoole_http_response类
        init_class_entry(swoole_http_response_ce, "swoole_http_response", swoole_http_response_methods)
        zend_register_internal_class(&swoole_http_response_ce)
        zend_register_class_alias(swoole_http_response,"swoole\\http\\response")

        init_class_entry(swoole_http_request_ce, "swoole_http_request", swoole_http_request_methods)
        zend_register_internal_class(&swoole_http_request_ce)
        zend_register_class_alias(swoole_http_request,"swoole\\http\\request")
    }

swoole_http_server->on(name, cb)

    //swoole_http_server.c:1047
    PHP_METHOD(swoole_http_server,on){
        //获取入参，name和cb
        zend_parse_parameters(ZEND_NUM_ARGS(), "zz", &event_name, &callback)

        //检查cb可执行性, 并获取callback 函数名
        zend_is_callable(callback,0, func_name)

        if event_name == Request{
            //给类增加一个属性，名称是onRequest, 值为用户创建的function，即：callback
            zend_update_property(swoole_http_server_class_entry_ptr, getThis(), "onRequest", callback);
            //注册到全局变量里面，里面报错者所有可以自定义的回调接口，即：onXXX系列
            php_sw_server_callbacks[SW_SERVER_CB_onRequest] = callback
        }

        if event_name == handshake {
            //基本逻辑通request
            //增加属性onHandshake,并赋值callback, 注册到callback 全局变量里面
            zend_update_property(swoole_http_server_class_entry_ptr, getThis(), "onHandshake", callback);
            php_sw_server_callbacks[SW_SERVER_CB_onHandshake] = callback
        }

        //onRequest 和 onHandshake是http server特有的，如果不是这2个中的一个，则调用父类的on接口，
        //该接口分析详见tcp_server
        else {
            zend_call_method_with_2_params(getThis(), swoole_server_class_entry_ptr, NULL, "on", &retval, event_name, callback)
        }
    }

swoole_http_server->start()

    //swoole_http_server.c:1295
    PHP_METHOD(swoole_http_server, start){
        //获取swServer, 该接口是从全局变量swoole_servers.array中根据object获取的，详见tcp server
        serv = swoole_get_object(getThis())
        //注册serv的各种回调接口供后续使用，详见tcp server有描述
        php_swoole_register_callback(serv)

        //检查改sock_type模式下必填回调是否设置，如果sock_type是websocket 则必须有onMessage
        //在这个地方检查websocket的原因是，swoole_http_server是swoole_websocket_server的父类
        php_sw_server_callbacks[onRequest]

        //各种初始化
        http_client_array, swoole_http_buffer, swoole_http_form_data_buffer, swoole_zlib_buffer

        //设置serv的onReceive 回调, 该接口描述见下
        serv->onReceive = http_onReceive

        zval *zsetting = zend_read_property(swoole_server_class_entry_ptr, getThis(), "setting")
        //初始化zsetting，并增加各种value
        array_init(zsetting)
        //key 包括，open_http_protocol, open_mqtt_protocol, openEof_check, open_length_check, open_websocket_protcol
        add_assoc_bool(zsetting, key, value)

        //根据zsetting的一些值来初始化 listen_list的属性
        serv->listen_list->open_http_protocol = 1
        ...

        //serv->ptr2都是存储的一个zval的object，ptr则是swoole_server的object
        serv->ptr2 = getThis()

        //for upload files
        zend_hash_init(rfc1867_upload_files,8, NULL, NULL,0)

        //2个很长很长，很复杂的函数，代码分析见tcp_server
        php_swoole_server_before_start(serv, getThis());
        swServer_start(serv)
    }
