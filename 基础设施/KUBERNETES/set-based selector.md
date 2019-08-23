# set-based selector

新版本的资源，就像`Job`，`Deployment`，`Replica_Set`，`Daemon_Set`，都支持set-based需求。

```yaml
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
```

