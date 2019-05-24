### 整理了下官方内容，发现一些坑所以整理给有需要的小伙伴，已亲自测试通过

### 使用方式
1. 安装 docker-compose 自行百度
2. 创建目录并设置权限
```
mkdir -p /data/mongodb/
mkdir -p /data/redisdb/
mkdir -p /data/easy-mock/logs
chmod 777 /data/easy-mock/logs
```
3. 运行 `vim /data/easy-mock/production.json`文件，并将下面的内容复制进入 `production.json`，退出并保存 `:wq`
```json
{
    "port": 7300,
    "host": "0.0.0.0",
    "pageSize": 30,
    "proxy": false,
    "db": "mongodb://mongodb/easy-mock",
    "unsplashClientId": "",
    "redis": {
        "keyPrefix": "[Easy Mock]",
        "port": 6379,
        "host": "redis",
        "password": "",
        "db": 0
    },
    "blackList": {
        "projects": [],
        "ips": []
    },
    "rateLimit": {
        "max": 1000,
        "duration": 1000
    },
    "jwt": {
        "expire": "14 days",
        "secret": "shared-secret"
    },
    "upload": {
        "types": [
            ".jpg",
            ".jpeg",
            ".png",
            ".gif",
            ".json",
            ".yml",
            ".yaml"
        ],
        "size": 5242880,
        "dir": "../public/upload",
        "expire": {
            "types": [
                ".json",
                ".yml",
                ".yaml"
            ],
            "day": -1
        }
    },
    "fe": {
        "copyright": "",
        "storageNamespace": "easy-mock_",
        "timeout": 25000,
        "publicPath": "/dist/"
    }
}
```
3. 新建文件 `docker-compose.yml` 并将下面的内容复制进入 `docker-compose.yml`。主要有三个需要替换的地方，数据库文件存储位置，日志文件存储位置，自定义配置文件本地地址。
```yaml
version: "3.3"
services:
  mongodb:
    image: mongo:3.4
    privileged: true
    volumes:
      - type: bind
        source: /data/mongodb # 数据库文件存放地址，根据需要修改为本地地址
        target: /data/db
  redis:
    image: redis:4.0.6
    privileged: true
    command: redis-server --appendonly yes
    volumes:
      - type: bind
        source: /data/redisdb # redis 数据文件存放地址，根据需要修改为本地地址
        target: /data
  web:
    image: easymock/easymock:1.6.0
    privileged: true
    command: /bin/bash -c "npm start"
    links:
      - mongodb:mongodb
      - redis:redis
    ports:
      - 7300:7300
    volumes:
      - type: bind
        source: /data/easy-mock/logs # 日志地址，根据需要修改为本地地址
        target: /home/easy-mock/easy-mock/logs
      - type: bind
        source: /data/easy-mock/production.json # 配置地址，请使用本地配置地址替换
        target: /home/easy-mock/easy-mock/config/production.json
```
4. 启动：`docker-compose up -d`

自定义配置参考 [easymock readme](https://github.com/easy-mock/easy-mock) 中的配置小节。

**注意**
* **使用容器方式运行不需要指定 `db` 和 `redis` 参数**
* **production.json 配置中注意以下问题**

```
{
  "port": 7300,
  "host": "0.0.0.0",
  "pageSize": 30,
  "proxy": false,
  "db": "mongodb://mongodb/easy-mock" # host 请务必替换为mongodb, 而非 localhost
  "unsplashClientId": "",
  "redis": {
    "keyPrefix": "[Easy Mock]",
    "port": 6379,
    "host": "redis", // 请勿使用 localhost，换 "redis"
    "password": "",
    "db": 0
  },
  ......
  ......
}
```

**异常问题**
* **常见错误处理：**
```text
server started at http://0.0.0.0:7300 events.js:182 throw er; // Unhandled 'error' event Error: EACCES: permission denied, open 'logs/2019-04-21-info.log' npm ERR! code ELIFECYCLE npm ERR! errno 1 npm ERR! easy-mock@1.6.0 start: cross-env NODE_ENV=production node app npm ERR! Exit status 1

解决办法 chmod 777 /home/*/logs
```

