# TIL

## RabbitMQ - msg_store_persistent: rebuilding indices from scratch

Happens after non-clean shutdown.

Fix if you don't care about queues and messages persistence:

    rm -rf /var/lib/rabbitmq/mnesia/<node name>/msg_store_persistent/*
    rm -rf /var/lib/rabbitmq/mnesia/<node name>/queues/*

## pip install lxml - gcc: internal compiler error: Killed (program cc1)

There is not enough memory.

Create a swap file:

    sudo dd if=/dev/zero of=/swapfile bs=1024 count=524288
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile


## PostgreSQL rows to JSON

    select row_to_json(t) from (select * from table) t;

## Celery - log task waiting time and execution time

    from celery.signals import before_task_publish, task_prerun, task_postrun

    @before_task_publish.connect
    def before_publish(*args, **kwargs):
        # Set timestamp in the message headers to calculate queue wait time later
        kwargs['headers']['before_publish_time'] = time.time()


    @task_prerun.connect
    def pre_run(sender, task_id, task, *args, **kwargs):
        task.pre_run_time = time.time()


    @task_postrun.connect
    def post_run(sender, task_id, task, retval, state,  *args, **kwargs):
        before_publish_time = task.request.headers.get('before_publish_time')
        pre_run_time = getattr(task, "pre_run_time", None)

        if before_publish_time and pre_run_time and state == 'SUCCESS':
            logger.info(
                "Task '{}' succeeded, waiting time: {}s, "
                "execution time: {}s".format(
                    task.name,
                    pre_run_time - before_publish_time,
                    time.time() - pre_run_time
                )
        )

## Unable to find venv/lib/python2.7/config/Makefile / easy_install error: None / distutils error: None

Get into that virtualenv and:

    python -m pip uninstall pip setuptools
    wget https://bootstrap.pypa.io/get-pip.py
    python get-pip.py
