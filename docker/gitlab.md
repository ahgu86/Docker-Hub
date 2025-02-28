### Docker部署自托管gitlab

官方镜像大约`3.4G`  基于`ubuntu:22.04`镜像构建

[官方仓库](https://gitlab.com/gitlab-org/omnibus-gitlab)

`docker-compose.yml`配置
```
services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    restart: always
    volumes:
      - './gitlab-data:/var/opt/gitlab'
      - './gitlab-config:/etc/gitlab'
      - './gitlab-logs:/var/log/gitlab'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://example.com'  # 设置域名

  caddy:
    image: caddy:alpine
    container_name: caddy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
    restart: always

```

`Caddyfile`配置

```
https://example.com {
    encode zstd gzip
    reverse_proxy gitlab:80

    # 启用 WebSocket 连接
    header {
        Upgrade        "websocket"
        Connection     "upgrade"
    }
}
```


默认用户名为：`root`

默认密码在`./gitlab-config/initial_root_password`文件里查看

更多个性化配置在`./gitlab-config/gitlab.rb`文件里修改
