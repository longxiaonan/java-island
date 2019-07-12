# docker错误集

### docker ps报错

执行docker命令后，如果没启动docker会导致报错`Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`

```shell
[root@test1 data]# docker ps
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
[root@test1 data]# service docker run
The service command supports only basic LSB actions (start, stop, restart, try-restart, reload, force-reload, status). For other actions, please try to use systemctl.
[root@test1 data]# service docker start
Redirecting to /bin/systemctl start docker.service
[root@test1 data]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

```

