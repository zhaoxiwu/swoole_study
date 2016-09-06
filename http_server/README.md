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
    //函数可以通过制定sock type类支持不同协议类型的服务
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

response->cookie()
    //swoole_http_server.c:1958
    PHP_METHOD(swoole_http_response,cookie){


    }

