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
