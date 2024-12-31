

### 交叉编译环境配置

编译linux上程序设置
```bash
set GOARCH=amd64
go env -w GOARCH=amd64
set GOOS=linux
go env -w GOOS=linux
```
切换回windows

```bash
go env -w GOARCH=amd64
go env -w GOOS=windows
```

