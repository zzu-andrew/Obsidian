







## 断网

```bash
--destination-ip string   目标 IP. 支持通过子网掩码来指定一个网段的IP地址, 例如 192.168.1.0/24. 则 192.168.1.0~192.168.1.255 都生效。你也可以指定固定的 IP，如 192.168.1.1 或者 192.168.1.1/32，也可以通过都号分隔多个参数，例如 192.168.1.1,192.168.2.1。
--exclude-port string     排除掉的端口，默认会忽略掉通信的对端端口，目的是保留通信可用。可以指定多个，使用逗号分隔或者连接符表示范围，例如 22,8000 或者 8000-8010。 这个参数不能与 --local-port 或者 --remote-port 参数一起使用
--exclude-ip string       排除受影响的 IP，支持通过子网掩码来指定一个网段的IP地址, 例如 192.168.1.0/24. 则 192.168.1.0~192.168.1.255 都生效。你也可以指定固定的 IP，如 192.168.1.1 或者 192.168.1.1/32，也可以通过都号分隔多个参数，例如 192.168.1.1,192.168.2.1。
--interface string        网卡设备，例如 eth0 (必要参数)
--local-port string       本地端口，一般是本机暴露服务的端口。可以指定多个，使用逗号分隔或者连接符表示范围，例如 80,8000-8080
--percent string          丢包百分比，取值在[0, 100]的正整数 (必要参数)
--remote-port string      远程端口，一般是要访问的外部暴露服务的端口。可以指定多个，使用逗号分隔或者连接符表示范围，例如 80,8000-8080
--force                   强制覆盖已有的 tc 规则，请务必在明确之前的规则可覆盖的情况下使用
--ignore-peer-port        针对添加 --exclude-port 参数，报 ss 命令找不到的情况下使用，忽略排除端口
--timeout string          设定运行时长，单位是秒，通用参数
```


```




- 断网 `600s` 排除端口 22  
```bash
./blade create network loss --percent 100 --interface eth0 --timeout 600 --exclude-port 22
```






## 内存占用


| --mem-percent string | 内存使用率，取值是 0 到 100 的整数                                                                                 |
| -------------------- | ----------------------------------------------------------------------------------------------------- |
| --mode string        | 内存占用模式，有 ram 和 cache 两种，例如 --mode ram。ram 采用代码实现，可控制占用速率，优先推荐此模式；cache 是通过挂载tmpfs实现；默认值是 --mode cache |
| --reserve string     | 保留内存的大小，单位是MB，如果 mem-percent 参数存在，则优先使用 mem-percent 参数                                                |
| --rate string        | 内存占用速率，单位是 MB/S，仅在 --mode ram 时生效                                                                     |
| --timeout string     | 设定运行时长，单位是秒，通用参数                                                                                      |


```bash
# 执行内存占用 50%
./blade c mem load --mode ram --mem-percent 50 -timeout 3600
```






















[chaosblade](https://chaosblade-io.gitbook.io/chaosblade-help-zh-cn/blade-create-mem-load)