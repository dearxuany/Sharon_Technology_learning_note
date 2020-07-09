# docker 使用阿里云容器镜像仓库
## 容器仓库用户权限设置
仓库权限文档 https://help.aliyun.com/document_detail/67992.html?spm=5176.8351553.0.0.63951991NhAXbN </br>
自定义权限策略允许访问仓库内某个项目镜像 AliyunContainerRegistryProjectFullAccess_ocr-server
```
{
    "Version": "1",
    "Statement": [
        {
            "Action": [
                "cr:*"
            ],
            "Effect": "Allow",
            "Resource": [
                "acs:cr:cn-shenzhen:*:repository/namespace/ocr-server"
            ]
        },
        {
            "Action": [
                "cr:Get*",
                "cr:List*"
            ],
            "Effect": "Allow",
            "Resource": [
                "acs:cr:*:*:repository/namespace"
            ]
        }
    ]
}
```
## 容器仓库登录
docker login 在主机设置登录阿里云容器镜像仓库
```
# docker login registry.cn-shenzhen.aliyuncs.com
Username: ops_acr@1244575467
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store


Login Succeeded
```
查看登录设置
```
#  cat ~/.docker/config.json
{
    "auths": {
        "registry.cn-hangzhou.aliyuncs.com": {
            "auth": "baaaaaaaasvsdv"
        }
    },
    "HttpHeaders": {
        "User-Agent": "Docker-Client/19.03.5 (linux)"
    }
```

## 镜像上传
本地镜像重命名并上传到仓库
```
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nvidia/cuda         10.0-base           841d44dd4b3c        4 months ago        110MB

# docker tag 841d44dd4b3c registry.cn-shenzhen.aliyuncs.com/namespace/ocr-server:10.0-base

# docker images
REPOSITORY                                                     TAG                 IMAGE ID            CREATED             SIZE
nvidia/cuda                                                    10.0-base           841d44dd4b3c        4 months ago        110MB
registry.cn-shenzhen.aliyuncs.com/namespace/ocr-server   10.0-base           841d44dd4b3c        4 months ago        110MB

# docker push registry.cn-shenzhen.aliyuncs.com/namespace/ocr-server:10.0-base
The push refers to repository [registry.cn-shenzhen.aliyuncs.com/namespace/ocr-server]
52ad947270f1: Pushed
dd841c774a30: Pushed
37b9a4b22186: Pushed
e0b3afb09dc3: Pushed
6c01b5a53aac: Pushed
2c6ac8e5063e: Pushed
cc967c529ced: Pushed
10.0-base: digest: sha256:e6e1001f286d084f8a3aea991afbcfe92cd389ad1f4883491d43631f152f175e size: 1781
```
## 镜像拉取
```
docker pull registry.cn-shenzhen.aliyuncs.com/namespace/ocr-server:10.0-base
```



