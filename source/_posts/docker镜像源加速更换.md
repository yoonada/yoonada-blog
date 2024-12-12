---
title: docker镜像源加速更换
date: 2024-03-13 10:38:41
tags: [docker]
categories: [docker] 
---
# docker镜像源加速更换

```sh
sudo tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "https://dockerproxy.com",
        "https://mirror.baidubce.com",
        "https://ccr.ccs.tencentyun.com",
        "https://docker.m.daocloud.io",
        "https://docker.nju.edu.cn",
        "https://docker.mirrors.ustc.edu.cn"
    ]
}
EOF
```

```sh
sudo systemctl daemon-reload
```

```sh
sudo systemctl restart docker
```


