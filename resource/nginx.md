**查看nginx所在目录**

```bash
brew list nginx
```

**配置文件修改地方**

```bash
/opt/homebrew/etc/nginx/nginx.conf
```

**使用多个.conf文件配置多个虚拟主机server**

> https://www.jianshu.com/p/dcdc7b73db6a

**查看当前ngnix使用配置位置以及该文件语法是否正确**

```bash
nginx -t
```

**查看nginx版本**

```bash
nginx -v
```

**重启nginx**

```bash
sudo nginx -s reload
```

**查看nginx服务状态**

```bash
sudo service nginx status
```

**停止nginx**

```bash
# 强行停止
sudo nginx -s stop
# 从容停止
sudo ngin -s quit
```

# 问题集合及处理

**nginx启动时报[error] invalid PID number ““ in “/usr/local/var/run/nginx/nginx.pid错误**

```bash
# 执行以下指令 指定特定的nginx配置文件
sudo nginx -c /usr/local/etc/nginx/nginx.conf
# 然后执行：
sudo nginx -s reload
```

