* # Ingress Nginx Controller

可用annotations

| Name | Descriptions |
| :--- | :--- |
| nginx.ingress.kubernetes.io/app-root | 不能完全理解其功用 |
| nginx.ingress.kubernetes.io/affinity | 唯一可用值“cookie”，基于cookie的负载 |
| nginx.ingress.kubernetes.io/affinity-mode | 可用值“balanced”负载均衡模式，“persistent”持久化模式（Stickyness of a session）,persistent将使得scale后的deployment不会再平衡。 |
| nginx.ingress.kubernetes.io/backend-protocol | 支持协议类型"HTTP","HTTPS","GRPC","GRPCS"和"AJP" |
| nginx.ingress.kubernetes.io/client-body-buffer-size | 一般情况下配置为两个内存页大小，64位系统，设置位16K即可Stickyness |
| nginx.ingress.kubernetes.io/configuration-snippet | 支持自定义向location中添加配置 |
| nginx.ingress.kubernetes.io/enable-cors | 是否允许跨域 |
| nginx.ingress.kubernetes.io/cors-allow-methods | 控制可接受的方法，默认为"GET ,PUT, DELETE, PATCH, OPTIONS" |
| nignx.ingress.kubernetes.io/cors-allow-headers | 可接受的Headers，默认：DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization |
| nginx.ingress.kubernetes.io/cors-allow-origin | 设置allow-origin |
| nginx.ingress.kubernetes.io/ssl-redirect | 默认当ingress的TLS开启，controller会重定向请求到https，可为单独的ingress添加annotation禁用重定向，也可从ConfigMap中配置，全局生效 |
| nginx.ingress.kubernetes.io/limit-connections | 限制单一IP最大并发连接数 |
| nginx.ingress.kubernetes.io/limit-rps |  |
| nginx.ingress.kubernetes.io/limit-rpm |  |
| nginx.ingress.kubernetes.io/permanent-redirect | 永久重定向308 |
| nginx.ingress.kubernetes.io/temporal-redirect | 临时重定向302 |
| nginx.ingress.kubernetes.io/proxy-body-size | 定义可接受的请求体大小，可配置ConfigMap使全局生效 |
| nginx.ingress.kubernetes.io/proxy-cookie-domain | 转换响应中Set-Cookie域中domain |
| nginx.ingress.kubernetes.io/proxy-cookie-path | 转换响应中Set-Cookie域中path |
| nginx.ingress.kubernetes.io/proxy-connect-timeout | TIMEOUT |
| nginx.ingress.kubernetes.io/rewrite-target | rewrite配置 |
| nginx.ingress.kubernetes.io/server-snippet | 自定义server配置段 |
| nginx.ingress.kubernetes.io/whitelist-source-range | 客户端源地址白名单（可测试实现切运维） |

另外，nginx-controller支持lua实现的waf策略，以提供服务的安全性保证。

