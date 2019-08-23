# Docker复盘-持久化

-v用于独立container，--mount用于swarm网络下的挂载

```
$ docker service create \
     --mount 'type=volume,src=<VOLUME-NAME>,\
     dst=<CONTAINER-PATH>,\
     volume-driver=local,\
     volume-opt=type=nfs,\
     volume-opt=device=<nfs-server>:<nfs-path>,\
     volume-opt=o=addr=<nfs-address>,vers=4,soft,timeo=180,bg,tcp,rw'
    --name myservice \
    <IMAGE>
```



