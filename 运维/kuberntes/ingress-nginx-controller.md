# Ingress Nginx Controller

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



