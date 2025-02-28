### 容器化定时任务

[项目地址](https://github.com/mcuadros/ofelia)

Ofelia是一款基于 Go 语言构建的现代、低资源占用的Docker环境作业调度程序。Ofelia 旨在取代老式的cron ⁠。

特点：
```
job-exec：在已经运行的容器里执行命令。
job-run：每次任务都启动一个新的容器来执行命令。
job-local：直接在 Ofelia 自己容器里执行命令。
job-service-run：在 Docker Swarm 集群中启动一个临时服务来执行命令。
```


#### 在别的容器里执行任务示例
`config.ini`配置示例
```
[job-exec "curl-task"]
# 每2秒执行一次
schedule = @every 2s
# 要执行的命令
command = curl -s 127.0.0.1
# 在nginx容器中执行
container = nginx
```

`docker`启动命令
```
docker run -d --name ofelia \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v ./ofelia.ini:/etc/ofelia/config.ini \
  --restart always \
  mcuadros/ofelia:latest
```
