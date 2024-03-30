---
title: "Docker容器内访问宿主机127.0.0.1(localhost)服务"
date: 2024-03-29T21:03:21+08:00
lastmod: ":fileModTime"
draft: false
---

# 情景再现

已经通过docker启动的[GitHub - danni-cool/wechatbot-webhook: http 请求驱动的微信机器人 (kkgithub.com)](https://kkgithub.com/danni-cool/wechatbot-webhook)，桥接模式，映射了3001端口。

在宿主机中直接可以测试推消息API

```shell
curl --location 'http://localhost:3001/webhook/msg/v2?token=[YOUR_PERSONAL_TOKEN]' \
--header 'Content-Type: application/json' \
--data '{
    "to": "testUser",
    "data": { "content": "你好👋" }
}'
```

根据[微信机器人运行在Linux服务器上？消息自动回复，Docker一键部署。详细安装使用 - 哔哩哔哩 (bilibili.com)](https://www.bilibili.com/read/cv28706223/)一文，改写一个python的web端来接收转发过来的消息。

```python
from fastapi import FastAPI, Form, UploadFile, File
import uvicorn

app = FastAPI()

@app.post("/receive_msg")
async def receive_msg(type: str = Form(...), content: str = Form(...), source: str = Form(None), is_mentioned: str = Form(None), is_msg_from_self: str = Form(None), file: UploadFile = File(None)):
    if type and content:
        # 根据消息类型执行相应操作
        if type == 'text':
            # 处理文本消息
            print('Received text message:', content)
        elif type == 'file':
            # 处理文件消息
            if file:
                # 处理文件上传逻辑
                print('Received file:', file.filename)
        elif type == 'urlLink':
            # 处理链接卡片消息
            print('Received URL link message:', content)
        # 其他消息类型的处理逻辑

        # 其他字段的处理
        if source:
            print('Source data:', source)

        if is_mentioned:
            print('Is mentioned:', is_mentioned)

        if is_msg_from_self:
            print('Is message from self:', is_msg_from_self)

        return {'message': 'Message received successfully'}

    return {'error': 'Invalid message format'}

if __name__ == '__main__':
    uvicorn.run(app, host="0.0.0.0", port=8000)

```

但是再看日志就出现报错：

```shell
[2024-03-29T12:26:02.984] [INFO] - starting fetching api: http://localhost:8000/receive_msg 
[2024-03-29T12:26:03.122] [ERROR] - Error occurred when trying to send Data to RecvdApi FetchError: request to http://localhost:8000/receive_msg failed, reason: connect ECONNREFUSED ::1:8000
    at ClientRequest.<anonymous> (/app/node_modules/.pnpm/node-fetch-commonjs@3.3.2/node_modules/node-fetch-commonjs/index.js:2223:11)
    at ClientRequest.emit (node:events:529:35)
    at Socket.socketErrorListener (node:_http_client:501:9)
    at Socket.emit (node:events:517:28)
    at emitErrorNT (node:internal/streams/destroy:151:8)
    at emitErrorCloseNT (node:internal/streams/destroy:116:3)
    at process.processTicksAndRejections (node:internal/process/task_queues:82:21) {
  type: 'system',
  errno: 'ECONNREFUSED',
  code: 'ECONNREFUSED',
  erroredSysCall: 'connect'
} 
```

# 原因分析

> docker是一个虚拟环境， `127.0.0.1`和`localhost`指的是虚拟环境内部，而不是外部宿主机，所以无法这样访问。

# 解决方案

1、 对于`mac`和`windows`（适用于 Docker Desktop 版本 18.03 及更高版本），使用`host.docker.internal`替换`127.0.0.1`,如`http://host.docker.internal:8000`

2、（未测试，自身设备无线连接，ip可能变动）对于Linux可以采用如下方案: 创建一个桥接网络

下面的`localNet`是网络名字,可自行修改;关于`192.168.0.0`这个子网,也可以自行定义.

默认按照下面的命令,执行后将可以通过`192.168.0.1`访问宿主机.

```undefined
docker network create -d bridge --subnet 192.168.0.0/24 --gateway 192.168.0.1 localNet
```

使用`192.168.0.1`替换`127.0.0.1`,如`http://192.168.0.1:3306`

3、使用容器的网络模式为"host"：在创建容器时，指定网络模式为"host"，容器将直接使用宿主机的网络命名空间，可以直接通过localhost访问宿主机的服务。这种方式适用于容器与宿主机共享网络资源的场景，但可能存在端口冲突的问题。

4、（测试成功）使用容器的IP地址：每个容器都有自己的IP地址，可以通过容器的IP地址来访问容器内部的服务。可以通过命令`docker inspect <容器ID>`获取容器的IP地址，然后在容器内部使用该IP地址访问localhost。

5、（未测试，自身设备无线连接，ip可能变动）使用宿主机的IP地址：可以通过宿主机的IP地址来访问宿主机上的服务。可以通过命令`ifconfig`或`ipconfig`获取宿主机的IP地址，然后在容器内部使用该IP地址访问localhost。

# 容器内测试

容器内比较精简，原有如下测试例子，没有curl能跑。

```shell
curl --location 'https://your.recvdapi.com' \
--form 'type="file"' \
--form 'content=@"/Users/Downloads/13482835.jpeg"' \
--form 'source="{\\\"room\\\":\\\"\\\",\\\"to\\\":{\\\"_events\\\":{},\\\"_eventsCount\\\":0,\\\"id\\\":\\\"@f387910fa45\\\",\\\"payload\\\":{\\\"alias\\\":\\\"\\\",\\\"avatar\\\":\\\"/cgi-bin/mmwebwx-bin/webwxgeticon?seq=1302335654&username=@f38bfd1e0567910fa45&skey=@crypaafc30\\\",\\\"friend\\\":false,\\\"gender\\\":1,\\\"id\\\":\\\"@f38bfd1e10fa45\\\",\\\"name\\\":\\\"ch.\\\",\\\"phone\\\":[],\\\"star\\\":false,\\\"type\\\":1}},\\\"from\\\":{\\\"_events\\\":{},\\\"_eventsCount\\\":0,\\\"id\\\":\\\"@6b5111dcc269b6901fbb58\\\",\\\"payload\\\":{\\\"address\\\":\\\"\\\",\\\"alias\\\":\\\"\\\",\\\"avatar\\\":\\\"/cgi-bin/mmwebwx-bin/webwxgeticon?seq=123234564&username=@6b5dbb58&skey=@crypt_ec356afc30\\\",\\\"city\\\":\\\"Mars\\\",\\\"friend\\\":false,\\\"gender\\\":1,\\\"id\\\":\\\"@6b5dbd3facb58\\\",\\\"name\\\":\\\"Daniel\\\",\\\"phone\\\":[],\\\"province\\\":\\\"Earth\\\",\\\"signature\\\":\\\"\\\",\\\"star\\\":false,\\\"weixin\\\":\\\"\\\",\\\"type\\\":1}}}"' \
--form 'isMentioned="0"'
```

搜索发现可以使用`nc`命令将就，前面是原始HTTP代码。

![Snipaste_2024-03-29_21-27-10](D:\program\blog\maniac.github.io\content\tutorials\docker\docker-localhost\Snipaste_2024-03-29_21-27-10.png)

宿主机接收成功

![Snipaste_2024-03-29_21-28-50](D:\program\blog\maniac.github.io\content\tutorials\docker\docker-localhost\Snipaste_2024-03-29_21-28-50.png)

后续再修改下处理程序

# 参考

[docker容器内访问宿主机127.0.0.1服务 - 简书 (jianshu.com)](https://www.jianshu.com/p/be6d5d71557d)
[从docker容器内部向localhost发出请求_使用nock从docker容器发出请求？_向Docker容器上的外部主机发出请求 - 腾讯云开发者社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/information/从docker容器内部向localhost发出请求-album)
