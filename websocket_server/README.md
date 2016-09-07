#WebSocket

demo:

    $serv = new swoole_websocket_server("127.0.0.1", 9502);

    $serv->on('Open', function($server, $req) {
        echo "connection open: ".$req->fd;
    });

    $serv->on('Message', function($server, $frame) {
        echo "message: ".$frame->data;
        $server->push($frame->fd, json_encode(["hello", "world"]));
    });

    $serv->on('Close', function($server, $fd) {
        echo "connection close: ".$fd;
    });

    $serv->start();


#webSocket_server()

    同swoole_http_server一样没有显示的构造函数，是在MINIT阶段通过调用swoole_websocket_init(module_number)实现
    swoole_websocket_init(module_number){
        init_class_entry(swoole_websocket_server_ce, "swoole_websocket_server", swoole_websocket_server_methods);
        //继承swoole_http_server
        zend_register_internal_class_ex(&swoole_websocket_server_ce, swoole_http_server_class_entry_ptr, "swoole_http_server");

        //定义frame类
        init_class_entry(swoole_websocket_frame_ce, "swoole_websocket_frame", NULL)
        zend_register_internal_class(&swoole_websocket_frame_ce)
    }

#swoole_websocket_server->on()

    PHP_METHOD(swoole_websocket_server,on){
        zend_parse_parameters(ZEND_NUM_ARGS(), "zz", &event_name, &callback)

        //获取swServer
        //根据zend object的handle从全局变量 swoole_objects.array[handle]中获取
        serv = swoole_get_object(getThis())

        //检查callback可执行性,逻辑同其他2个server
        zend_is_callable(callback, 0, &func_name TSRMLS_CC)

        //根据event_name 注册到swoole_websocket_server_class_entry_ptr不同的属性中去
        //webSocket_server 独有的2个回调是onOpen, onMessage
        //例子只分析其中一种
        if event_name == "open"{
            zend_update_property(swoole_websocket_server_class_entry_ptr, getThis(), "onOpen", callback)
            php_sw_server_callbacks[SW_SERVER_CB_onOpen] = callback
        }

        //如果不是open和message中的一种，则调用父类的on方法
        else{
            zval *obj = getThis()
            zen_call_method_with_2_param(&obj, swoole_http_server_class_entry_ptr, NULL, "on",&return_value,event_name, callback)
        }
    }

#swoole_websocket_server->push

    PHP_METHOD(swoole_websocket_server, push){
        //fd, zdata, opcode, fin
        zend_parse_parameters(ZEND_NUM_ARGS, "lz|lb", &fd, &zdata, &opcode, &fin)

        //opcode must smaller then 10

        //get data from zdata
        php_swoole_get_send_data(zdata, &data){
            //zval to char
        }

        //get connection to send data, 根据fd获取connection
        swConnect *conn = swWorker_get_connection(SwooleG.serv, fd){
            serv->connection_list[fd]
        }

        //清空buffer
        swString_clear(swoole_http_buffer)

        //对数据进行打包encode
        swWebSocket_encode(swoole_http_buffer, data, length, opcode, fin, 0){
            //TODO
        }

        swServer_tcp_send(SwooleG.serv, fd, swoole_http_buffer->str, swoole_http_buffer->length){
            factory = serv->factory
            send.info.d = fd
            send.info.type = SW_ENENT_TCP
            send.data = data //data = swoole_http_buffer->str
            factory->finish(factory, send)
        }
    }


#swoole_websocket_server->pack

    PHP_METHOD(swoole_websocket_server, pack){
        zend_parse_parameters(ZEND_NUM_ARGS, "s|lbb", &data, &length, &opcode, &finish, &mask)
        //clear http buffer
        swString_clear(swoole_http_buffer)

        //encode input data
        swWebSocket_encode(swoole_http_buffer, data, length, opcode, finish, mask){
    }
#swoole_websocket_server->unpack

    PHP_METHOD(swoole_websocket_server, unpack){
        zend_parse_parameters(ZEND_NUM_ARGS, "s", &data)
        php_swoole_websocket_unpack(data)
    }
#swoole_websocket_server->exist

    PHP_METHOD(swoole_websocket_server, unpack){
        //存在性检查，根据fd查看对应的connection 和 server_sock是否存在
    }
