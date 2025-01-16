








- 查询当前节点状态
```bash
./etcdctl --endpoints=$ENDPOINTS endpoint status --write-out=table
# 使用示例
 etcdctl  --endpoints=10.161.48.46:12256,10.161.48.45:12256 endpoint status --write-out=table
```