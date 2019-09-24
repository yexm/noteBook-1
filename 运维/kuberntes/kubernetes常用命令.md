# kubernetes 常用命令

## 监控

### Kubernetes POD内存使用排序Top30

```bash
kubectl top pod|sort -k3 -n -r|head -30
```

稍作修改 sort -k2 按照CPU使用总时间排序