#Task

demo
    $serv = new swoole_server("127.0.0.1", 9502);
    $serv->set(array('task_worker_num' => 4));
    $serv->on('Receive', function($serv, $fd, $from_id, $data) {
        $task_id = $serv->task("Async");
        echo "Dispath AsyncTask: id=$task_id\n";
    });
    $serv->on('Task', function ($serv, $task_id, $from_id, $data) {
        echo "New AsyncTask[id=$task_id]".PHP_EOL;
        $serv->finish("$data -> OK");
    });
    $serv->on('Finish', function ($serv, $task_id, $data) {
        echo "AsyncTask[$task_id] Finish: $data".PHP_EOL;
    });
    $serv->start();

#serv->task()

    PHP_METHOD(swoole_server, Task){
        zend_parse_parameters(ZEND_NUM_ARGS, "z|lz", &data, &dst_worker_id, &callback)

        //各种检查，如果未指定dst_worker_id，则随机分配一个
        php_swoole_check_task_param(dest_worker_id)
        //调用zend引擎的serialize
        php_swoole_task_pack(&buf,data)
        //callback 可执行性检查
        zend_is_callable(callback,0,&func_name)

        swProcessPool_dispatch(&SwooleGS->task_workers, &buf, dest_worker_id){
            if dest_worker_id < 0 {
                dest_worker_id = swProcess_schedule(pool){
                    //TODO
                    if pool->dispatch_mod == SW_DISPATCH_QUEUE{
                        return 0
                    }

                    for(i;i<run_worker_num;i++){
                        //根据round_id的值模总worker个数，查看该worker是否空闲，空闲则返回worker_id
                        worker_id = sw_atomic_fetch_add(pool->round_id, 1)%run_worker_num
                        if poole_worker[worker_id].status == SW_WORKER_IDLE {
                            return worker_id
                        }
                }
            }

            swWorker_send2worker(worker,data,sendn, SW_PIPE_MASTER|SW_PIPI_NONBLOCK){
                //TODO
                if flag & SW_PIPE_MASTER{
                    pipefd = worker->pipe_master
                }

                if worker->pool->use_msgqueue{
                    swMsgqueue_push(queue, data, length){
                        msgsnd()
                    }
                }

                if NONBLOCK && main_reactor{
                    swooleG.main_reactor->write()
                }

                else{
                    swSocket_write_blocking(pipefd, buf, n){
                        //系统调用
                        write(pipefd, buf, n)
                    }
                }
            }
            sw_atomic_fetch_add(&worker->tasking,1)

        }
        //增加计数
        sw_atomic_fetch_add(&SwooleStats->tasking_num, 1);
    }


