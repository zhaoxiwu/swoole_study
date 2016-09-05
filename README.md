# swoole_study
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
demo:

    // create a server instance
    $serv = new swoole_server("127.0.0.1", 9501);

    // attach handler for connect event, once client connected to server the registered handler will be executed
    $serv->on('connect', function ($serv, $fd){
        echo "Client:Connect.\n";
    });

    // attach handler for receive event, every piece of data received by server, the registered handler will be
    // executed. And all custom protocol implementation should be located there.
    $serv->on('receive', function ($serv, $fd, $from_id, $data) {
        $serv->send($fd, $data);
    });

    $serv->on('close', function ($serv, $fd) {
        echo "Client: Close.\n";
    });

    // start our server, listen on port and ready to accept connections
    $serv->start();


--------------------------------------------------------------------------------
PHP用户空间：
    swoole_websock::swoole_http_server::swoole_server,

    swoole_server(string  $host, integer  $port, integer  $mode = SWOOLE_PROCESS, integer  $sock_type = SWOOLE_SOCK_TCP)


    integer $mode SWOOLE_BASE, SWOOLE_THREAD, SWOOLE_PROCESS, SWOOLE_PACKET

    integer $sock_type  SWOOLE_SOCK_TCP, SWOOLE_SOCK_TCP6, SWOOLE_SOCK_UDP, SWOOLE_SOCK_UDP6, SWOOLE_SOCK_UNIX_DGRAM, SWOOLE_SOCK_UNIX_STREAM,
     If you want use ssl just or (|) your current socket type with SWOOLE_SSL


swoole_server(serv_host,serv_port,serv_mode,sock_type)

    创建过程：
    1、启动方式检查，只支持cli
    2、重复创建检查，只允许有一个server
    3、启动状况检查，运行中不能创建server
    4、swServer_init（serv）{
         swoole_init(){
             初始化全局变量swooleG, swooleWG
             设置swoolG.log_fd, cup_num,pagesize,pid,memory_pool(swMemoryGlobal),
             初始化swooleGS自身,内部的锁
             获取系统rlimit，并设置swooleG.max_sockets
             初始化SwooleStats
         }
         serv->factory_mod = SW_MODE_BASE
         serv->reactor_num
         serv->dispatch_mod = SW_DISPATCH_FDMOD
         serv->ringbuffer_size = SW_QUEUE_SIZE
         serv->wormer_num, max_connection, max_request, http_parse_post, heartbeat check, buffer size
         swooleG.serv = serv
    }
    5、serv_mod 不支持SW_MODE_THREAD , SW_MODE_BASE
    6、//创建一个socket并加入到serv的listen_list中维护起来
    swListenPort *port = swServer_add_port(serv, sock_type, serv_host, serv_port){
              端口号检查：端口号范围，多端口监听个数限制
              swListenPort *ls 初始化
              ls->type=sock_type, ls->port=serv_port, ls->ssl=1/0,
              int sock = swSocket_create(ls_type){
                  //domain取决于ls_type的值，创建一个linux的socket
                   socket(domain, ls_type);
              }

              swSocket_bind(sock, ls->type, ls->host, ls->port){
                  //系统调用bind， host和port设置到sockaddr中
                   bind(sock, sockaddr, sizeof(sockaddr))
              }

              //ls->type 如果是TCP, TCP6, STREAM中的一种则设置为noblock
              ioctl(sock, FIONBIO, &nonblock);
              ls->sock=sock
              serv->have_tcp_dock = 1/0
              //serv->listen_list 是一个双向链表，每个元素有prev，next
              append(serv->listen_list, ls)
              serv->listen_port_num++
              return ls
         }
     7、swoole_objests[this.handle]=serv, server_port_list[this.handle]=port

    swoole_server.c:1746
    swoole->on(name, callback){
         server启动后不允许注册cb
         //cb check, 改接口为zend提供，如果cb为可执行，则func_name为接口名称
         zend_is_callable(cb, 0, &func_name TSRMLS_CC)
         //name转换为string类型，zend提供的接口
         convert_to_string(name);
         //cb 允许范围
         char *callback_name[PHP_SERVER_CALLBACK_NUM] = {
        ¦   "Connect",
        ¦   "Receive",
        ¦   "Close",
        ¦   "Packet",
        ¦   "Start",
        ¦   "Shutdown",
        ¦   "WorkerStart",
        ¦   "WorkerStop",
        ¦   "Task",
        ¦   "Finish",
        ¦   "WorkerError",
        ¦   "ManagerStart",
        ¦   "ManagerStop",
        ¦   "PipeMessage",
        ¦   NULL,
        ¦   NULL,
        ¦   NULL,
        ¦   NULL,
        };

        php_sw_server_callbacks[name]=cb
        if(name in (connect, receive,close,packet,start)){
             zval port_object = server_port_list.zobjects[0];
              //调用server_port_class 的on方法，参数为name和cb, 主要作用是给swooleG.server->onConnect/onClose设置回调函数
             zend_call_method_with_2_params(port_object, swoole_server_port_class_entry_ptr,null,”on",retval,name,cb)
         }
    }

    swoole_server_port.c:317
    swoole_server_port->on（name, cb）{
         server运行中不允许调用
         //检查cb的可执行性
         zend_is_callable(cb,0,&func_name)
         swoole_server_port_property *property = swoole_get_property(getThis(), 0){
              handle=Z_OBJ_HANDLE(getThis())
              swoole_objects.property[0][handle];
         }
         proterty_name=“on”+name
         swoole_server_port_class_entry->proterty_name=cb
         //注册上用户自定义cb，供后续onXXX系列函数调用
         proterty->callbacks[i]=cb
         //php_swoole_onConnect(swServer *serv, swDataHead *info),主要作用是调用用户注册的connect 回调函数
         // functsion($serv , $fd)
         swooleG.serv->onConnect = php_swoole_onConnect
         //同onConnect，调用用户注册的close函数，函数原型function($serv, $fd)
          swooleG.serv->onClose = php_swoole_onClose
    }

    swoole_server.c::1332
    php_swoole_onConnect/onClose(sw server 8serv, swDataHead *info){
         //设置调用参数, 其中fd为reactor与work交互的文件描述符，from_id为reactor id
         args[0]=&serv->ptr2, arg[1]=info.fd, arg[2]=info.from_id

         callback = php_swoole_server_get_callback(serv, info->form_fd, SW_SERVER_CB_onConnect/Close){
              swListenPort *port = serv->connection_list[info->form_fd].object
              return callback = port->ptr->callbacks[SW_SERVER_CB_onConnect]
         }
         //该接口由zend提供
         call_user_function_ex(EG(function_table), NULL, callback, retval,3,args,0,null)
    }

    swoole_server.c:1931
    swoole_sever->start(){
         运行状态检查
         swServer *serv = swoole_objects[this.handle]
         php_swoole_regisetr_callback(serv){
              //设置swServer的各类onXX接口的回调函数，其中onShutdown和onWorkerStart不可以用户自定义
              serv->onStart = php_swoole_onStart
              serv->onShutdown = php_swoole_onShutdown
              serv->onWorkerStart = php_swoole_onWorkerStart
              serv->onWorkerStop = php_swoole_onWorkerStop
              serv->onPacket = php_swoole_onPacket
              serv->onTask = php_swoole_onTask
              serv->onFinish = php_swoole_onFinish
              serv->onWorkerError = php_swoole_onWOrkerError
              serv->onManagerStart = php_swoole_onManagerStart
              serv->onManagerStop = php_swoole_onMangerStop
              serv->onPipeMessage = php_swoole_onPipeMessage
         }

         //不同sock type 注册的不一样。udp是onPacket
         onReceive和onPacket 2个回调接口必须有一个
         serv->onReseive = php_swoole_onReceive
         serv->ptr2 = zobject（zobject意思是zval object，serv是swServer。其实是swoole_server类的实例）

         //下面是一个很长的接口， 从名字可以看出是启动之前的一系列前置操作
         //整体逻辑为：设置serv和factory的关联关系，根据factory_mod的值创建不同的worker
         //初始化serv的reactor_threads,connection_list, 设置factory的各种回调接口
         //给server增加一个setting属性，用来存储配置相关的属性，如果worker_num等，并调用set接口
         //serv->set(array)是提供给用户来设置server属性的操作接口
         //swoole_server.c::322
         php_swoole_server_before_start(serv, zobject){
           swServer_create(serv){
             swooleG.factory = &serv->factory
             serv->factory.ptr = serv
             初始化serv->session_list
             if serv->factory_mod == SW_MODE_SINGLE{
               //进程模式启动，此时reactor和worker为同一个进程
               swReactorProcess_create(serv)
             }
             else{

               //当swoole以server模式运行时，reactor都是多线程启动，同时拥有多个worker，和taskWorker
               //src/network/ReactorThread.c:918
               swReactorThread_create(serv){
                 根据serv->reactor_num申请对应大小的内存
                 serv->reactor_threads = swooleG.memory_pool.alloc(memory_pool,serv->reactor_num*sizeof(swReactorThread))
                 //serv->factory_mod == SW_MODE_PROCESS时使用共享内存，
                 初始化serv->connection_list = sw_calloc/shm_calloc()
                 //接下来创建worker进程
                 SW_MODE_THREAD模式废弃
                 if serv->factory_mod == SW_MODE_PROCESS{
                   //src/factory/FactoryProcess.c::30
                   //第二个参数work_num 没有用到
                   swFactoryProcess_create(&serv->factory, serv->worker_num){
                     init swFactoryProcess *object
                     //set callbacks
                     factory->object = object
                     factory->dispatch = swFactoryProcess_dispatch
                     factory->finish = swFactoryProcess_finish
                     factory->start = swFactoryProcess_start
                     factory->notify = swFactoryProcess_notify
                     factory->Shutdown = swFactoryProcess_shutdown
                     factory->end = swFactoryPrcess_end
                   }
                 }
                 //默认情况下使用,
                 swFactory_create(&(serv->factory)){
                   同swFactoryProcess_create, 只是没有设置factory的object
                 }
               }
             }
           }
           //if use coroutien, 暂时先不看，后续补充 TODO
           coro_init()

           //设置swoole_server的属性master_pid的值
           zend_update_property_long(swoole_server_port_class_entry_ptr,zobject,"master_pid",getpid())

           init swoole_server属性setting，
           //给zsetting中增加属性和值, setting是一个zend_array,
           //属性值包括：worker_num, task_worker_num, pipe_buffer_size,buffer_output_size,max_connection
           //以worker_num为例
           add_assoc_long(zsetting, "worker_num", serv->worker_num)

           //循环调用 用户自定义set函数
           for（i=1;i<server_port_list.num;i++）{
             port_object = server_port_list.zobjects[i]
             //获取上面设置好的 setting
             port_setting = zend_read_property(class,object,"setting",1)
             zend_call_method_with_1_params(port_object, swoole_server_port_class_entry_ptr, null,"set",retval,zsetting)

           }
           //至此这个before start 任务完成
         }

         //src/network/server.c::502
         //swServer从这个地方正式启动
         //又是一个很长的接口
         swServer_start(serv){
          //获取server的factory, 这个家伙其实不是工厂模式
          //factory是manager进程，worker，taskWorker进程的总入口， 这些进程均在factory中fork
          *factory = &serv->factory

          //server 启动前的各种检查
          swServer_start_check(serv){
            //check onReceive, onPacket,have_tcp_sock是否符合规则
            //!udp
            serv->onPacket = serv->onReceive

            if serv->factory_mod == SW_MODE_PROCESS {
              //close notify
              if serv->dispatch_mod == SW_DISPATCH_ROUND/QUEUE
                serv->onConnect 和 serv->onClose = NULL
                serv->disable_notify=1
            }

            //onTask是用户主动把一些长耗时的请求分给taskWorker执行时的接口
            //onFinish为taskWorker执行完用户指派的任务后的回调接口
            if swooleG.task_worker_num{
              serv->onTask/onFinish 存在性检查
            }

            check serv->reactor_num/worker_num/max_sockets/max_connection 的合法性
            swooleG->session_round=1
          }

          //init message QUEUE
          //ftok convert a pathname and a project identifier to a System V IPC key
          serv->message_queue_key = ftok(path_ptr, 1);

          //TODO
          init log

          if serv->demonize{
            redirect stdout to log file
            redirect stdout stderr to /dev/null
          }

          //set master pid
          swooleGS->master_pid=getpid()
          swooleGS->start=1
          swooleGS->now = swooleGS->start_time = time(NULL)

          //设置send回调函数
          if udp{
            //TODO
            serv->send = swServer_send2
          }else{
            //TODO
            serv->send = swServer_send1
          }

          //alloc memory for workers
          serv->workers = swooleG.memory_pool->alloc(worker_num)

          //设置swooleGS 的 event_workers
          swooleGS->event_workers.workers = serv->workers
          swooleGS->event_workers.worker_num = serv->worker_num
          swooleGS->event_workers.use_msgqueue = 0

          //循环设置每个worker的pool
          for(){
            swooleGS->event_workers.workers[i].pool = &swooleGS->event_workers
          }

          //循环设置reactor thread的 buffer input
          for(){
            serv->reactor_threads[i].buffer_input = new swRingBuffer_new()
          }

          //循环设置task_worker的notify pipe 和result TODO

          //TODO user_worker 不知道干嘛的

          //循环设置serv->listen_list中ls的options
          for(listen_list as ls){
            //TODO
            swPort_set_option(ls);
          }

          //factory start
          //又是一个很复杂的函数，进去看看
          //用来创建worker和task worker
          //swServer_create中曾调用swFactoryProcess_create对factory的各种回调接口设置过
          //factory->start = swFactoryProcess_start
          //src/factory/FactoryProcess.c::68
          factory->start(factory){


          }

          //至此SW_SERVER_CB_onStart 结束  
         }
    }
