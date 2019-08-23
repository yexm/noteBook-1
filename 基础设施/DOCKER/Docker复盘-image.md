# Docker复盘-image

## DockerFile最佳实践

创建镜像的方法：

```
BuildContext
    使用dockerfile，默认情况下在buildcontex目录下默认加载dockerfile，可使用-f参数，指定dockerfile

使用pipe在stdin中生成image
    docker build -t imagename . -f- <<EOF
    ...
    EOF
```

17.05版本以上，建议利用多级构建机制减小镜像大小，尚未详细研究其核心原理 ，但官方给出dockerfile编写原则如下：

1. 安装构建应用所需的工具；
2. 安装依赖库；
3. 生成应用。

以上几条建议遵循着“先定义相对稳定的layer，在定义变化频繁的layer“这样的规律。

```
FROM golang:1.7.3 as builder
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html  
COPY app.go    .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /go/src/github.com/alexellis/href-counter/app .
CMD ["./app"]
```

#### Stop at a specific build stage

```
$ docker build --target builder -t alexellis2/href-counter:latest .
```

#### Use an external image as a “stage”

```
COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```

### Dockerfile 要点

* FROM
* LABEL
* RUN，实际上默认调用了bash -c， 可以通过\["CMD", "PARM1" ...\]，更改shell类型
* CMD，两个用途没有ENTRYPOINT的情况下，将提供image默认进程，可被docker run命令中的指令覆盖；有ENTRYPOINT的情况下，CMD将为其提供变量
* EXPOSE
* ENV
* ADD or COPY，更建议使用COPY，ADD作为一种对于压缩格式和URL的支持
* ENTRYPOINT
* VOLUME，建议对image的任何可变内容挂在volume
* USER
* WORKDIR



