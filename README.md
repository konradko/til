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


## Python decorator for logging uncaught exceptions and sending them to Sentry

    from raven import Client
    
    // Change this to your Sentry client key
    sentry_dsn = ''
    
    def send_uncaught_exceptions_to_sentry(func):
        """
        Decorator that sends exceptions to app.getsentry.com
        """
        def wrapped(*args, **kwargs):
            try:
                func(*args, **kwargs)
            except:
                if sentry_dsn:
                    # prints traceback
                    logging.exception("Exception occurred, sending to Sentry:")
                    Client(dsn=sentry_dsn).captureException()
                else:
                    raise
        return wrapped

    // Usage:

    @send_uncaught_exceptions_to_sentry
    def test():
        raise Exception('test')


## sed: -e expression #1, char 13: unknown option to `s'

The variable used in expression has slashes in it, change sed delimiter e.g.

    export EXAMPLE_PATH=/tmp/
    echo 'replace this' > file
    sed -i "s/replace this/$EXAMPLE_PATH/" example.txt
    sed: -e expression #1, char 13: unknown option to `s'

Change to:
    export EXAMPLE_PATH=/tmp/
    echo 'replace this' > file
    sed -i "s#replace this#$EXAMPLE_PATH#" example.txt
    cat file
    /tmp/


## Load process (e.g. docker container) environment variables
    
    while read -d $'\0' ENV; do declare "$ENV"; done < /proc/<pid>/environ

## [Sending HTTP requests concurrently with Go](https://gist.github.com/konradko/1a8a6b09b2272163c4c971a686760cb7)

## IPython logger

    logger = logging.getLogger()
    logger.addHandler(logging.StreamHandler())
