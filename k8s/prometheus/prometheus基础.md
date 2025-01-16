
外部架构图
![](attachments/architecture.svg)

内部架构图

![](attachments/internal_architecture.svg)





## 信息查看



### 查看配置

通过访问网址 : `ip:port/config` 即可查看启动状态下的 `prometheus` 的配置

```bash
http://10.161.40.240:30090/config
```









- kube state metrics 指标

```plaintext
sum(kube_pod_container_status_restarts_total) by (pod)
sum(kube_pod_status_phase{phase!="Running"}) by (namespace, phase)
```
