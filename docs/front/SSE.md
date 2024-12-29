---
sticky: 999
description: 10分钟完成自己的memos部署
descriptionHTML: '
<span style="color:var(--description-font-color);">10分钟完成自己的memos部署</span>
<pre style="background-color: #292b30; padding: 15px; border-radius: 10px;" class="shiki material-theme-palenight"><code>
    <span class="line"><span style="color:#FFCB6B;">npm</span><span style="color:#A6ACCD;"> </span><span style="color:#C3E88D;">create</span><span style="color:#A6ACCD;"> </span><span style="color:#C3E88D;">@sugarat/theme@latest</span></span>
    <br/>
    <br/>
    <span class="line"><span style="color:#B392F0;">bun create</span><span style="color:#E1E4E8;"> </span><span style="color:#9ECBFF;">@sugarat/theme</span><span style="color:#E1E4E8;"> </span></span>
</code>
</pre>'
tag:
  - 流式传输
top: 1
sidebar: true
date: 2024-12-30 23:40:00
---

# SSE

## 什么是SSE

Server-Sent Events (SSE) 是一种允许服务器向客户端推送实时更新的技术。与传统的客户端轮询（polling）不同，SSE 允许服务器在有新数据时主动将数据推送到客户端，而不需要客户端不断地向服务器请求数据。

SSE 与 WebSocket 作用相似，都是建立浏览器与服务器之间的通信渠道，然后服务器向浏览器推送信息。

参考文档：

https://html.spec.whatwg.org/multipage/server-sent-events.html 

https://www.ruanyifeng.com/blog/2017/05/server-sent_events.html

#### SSE的本质

HTTP协议无法做到服务器主动推送信息。但是，有一种变通方法，就是服务器向客户端声明，接下来要发送的是流信息（streaming）。

也就是说，发送的不是一次性的数据包，而是一个数据流，会连续不断地发送过来。这时，客户端不会关闭连接，会一直等着服务器发过来的新的数据流，视频播放就是这样的例子。本质上，这种通信就是以流信息的方式，完成一次用时很长的下载。

SSE 就是利用这种机制，使用流信息向浏览器推送信息。它基于 HTTP 协议，目前除了 IE/Edge，其他浏览器都支持。

#### 主要特点

1. **单向通信：** SSE是单向的，即服务器可以向客户端发送数据，但是客户端不能通过SSE向服务器发送数据，如果需要双向通信，通常会使用WebSocket。
2. **基于HTTP：** SSE是基于标准的HTTP协议，因此不需要额外的协议支持，客户端通过普通的HTTP请求与服务器建立连接，服务器通过这个连接持续发送数据。而WebSocket 是一个独立协议。
3. **自动重连：** 如果连接中断，SSE会默认自动尝试重新连接服务器。这使得SSE在处理网络不稳定的情况下非常有用。而WebSocket 需要自己实现。
4. **轻量级：** 简单易用，SSE的api相对简单，客户端可以使用EventSource对象来接收服务器推送的事件。
5. **支持自定义：** 支持自定义发送的消息类型。

#### SSE数据格式

在 SSE中，data: 是消息字段的标识符。SSE 协议规定，每条消息必须以 data: 开头，后面跟随消息内容。客户端通过解析 data: 来识别消息的有效载荷。

##### 消息格式

- 每条消息由多个字段组成，每个字段以字段名开头，后面跟随字段值。即：[field]: value\n
- 常见的字段包括 data:（消息内容）、event:（事件类型）、id:（消息ID）、retry等。
- 此外，还可以有冒号开头的行，表示注释。通常，服务器每隔一段时间就会向浏览器发送一个注释，保持连接不中断。例如   : This is a comment
- 每个字段以换行符 \n 结束。
- 一条完整的消息以两个换行符 \n\n 结束。

##### data 字段

数据内容用`data`字段表示。

```
data:  message\n\n
```

如果数据很长，可以分成多行，最后一行用`\n\n`结尾，前面行都用`\n`结尾。

```
data: begin message\n
data: continue message\n\n
```

下面是一个发送 JSON 数据的例子。

```
data: {\n
data: "foo": "bar",\n
data: "baz", 555\n
data: }\n\n
```

#### SSE流式模拟-基础

使用node来模拟SSE流式请求，分为纯流式输出文本或流式输出时带上附加信息：

```javascript
const express = require('express')
const cors = require('cors')

const app = express()

app.use(cors({ origin: '*', optionsSuccessStatus: 200 }))

app.use(express.json({ limit: '10mb' }))

// 设置 SSE 路由
app.all('/api/chat/sse', (req, res) => {
    console.log('请求参数', req.body)

    // 设置SSE头信息
    res.setHeader('Content-Type', 'text/event-stream; charset=utf-8')
    res.setHeader('Cache-Control', 'no-cache, no-transform')
    res.setHeader('Connection', 'keep-alive')
    res.setHeader('Access-Control-Allow-Origin', '*')
    res.setHeader('X-Accel-Buffering', 'no')

    // 发送初始数据
    res.write('data: Initial message\n\n')

    const markdownTextOne = `我是第一行文本`
    const markdownTextTwo = `# 这是一个标题
    ## 这是一个子标题
    这是一段100字的文本，用于演示SSE流式输出。我们将逐步发送这些字符，模拟实时数据更新。SSE是一种强大的技术，允许服务器向客户端推送数据，而不需要客户端频繁轮询。`

    const markdownText = req.body.type === 1 ? markdownTextOne : markdownTextTwo

    // 注意结尾\n 和\n\n的区别，\n代表换行，\n\n代表一条消息的结束

    let index = 0
    // 每10毫秒发送5个字符
    const interval = setInterval(() => {
        if (index < markdownText.length) {
            // 输出纯内容
            const chunk = markdownText.slice(index, index + 5)

            // 输出内容可以选择带其他内容
            // const chunk = JSON.stringify({
            //     content: markdownText.slice(index, index + 5),
            //     qId: 123,
            //     aId: 456,
            // })

            // 一定要加\n\n，代表这是一条完整的消息
            res.write(`data: ${chunk}\n\n`)
            index += 5
        } else {
            // 结束流式输出
            clearInterval(interval)
            res.end()

            // 结束返回时是否再输出内容
            // res.end(`data: 输出结束\n\n`) // 一定要加\n\n
        }
    }, 10)

    // 当客户端关闭连接时会触发
    req.on('close', () => {
        console.log('客户端关闭sse连接')
    })
})

// 启动服务
const PORT = process.env.PORT || 3000

app.listen(PORT, () => {
    console.log(`Server listening on port ${PORT}`)
})
```

##### 效果预览

按照上面代码配置完成，并确保SSE服务是正常的，即可在请求接口后看到如下效果。

![image-20241227145525604](/Users/admin/Library/Application Support/typora-user-images/image-20241227145525604.png)

#### SSE流式模拟-复杂

在某些业务场景下，SSE输出消息的同时，可能还需要输出id、类型等其他的参数来满足业务需求，参考如下代码。

```javascript
const express = require('express')
const cors = require('cors')

const app = express()

app.use(cors({ origin: '*', optionsSuccessStatus: 200 }))

app.use(express.json({ limit: '10mb' }))

app.all('/api/chat/sse', (req, res) => {
    console.log('请求参数', req.body)

    // 设置SSE头信息
    res.setHeader('Content-Type', 'text/event-stream; charset=utf-8')
    res.setHeader('Cache-Control', 'no-cache, no-transform')
    res.setHeader('Connection', 'keep-alive')
    res.setHeader('Access-Control-Allow-Origin', '*')
    res.setHeader('X-Accel-Buffering', 'no')

    // 发送初始数据
    res.write('data: Initial message\n\n')

    const markdownTextOne = `我是第一行文本`
    const markdownTextTwo = `# 这是一个标题
    ## 这段文本用来演示SSE如何输出。我们将逐步发送这些字符，模拟实时数据更新。SSE是一种强大的技术，允许服务器向客户端推送数据，而不需要客户端频繁轮询。`

    const markdownText = req.body.type === 1 ? markdownTextOne : markdownTextTwo

    let index = 0
    let messageId = 1

    // 每10毫秒发送5个字符
    const interval = setInterval(() => {
        if (index < markdownText.length) {
            const chunk = markdownText.slice(index, index + 5)

            res.write(': 这是一条注释\n')
            res.write(`id: ${messageId}\n`)
            res.write('event: content\n')
            res.write(`data: ${chunk}\n\n`)
            messageId++
            index += 5
        } else {
            // 结束流式输出
            clearInterval(interval)
            res.write(`id: ${messageId}\n`)
            res.write('event: end\n')
            res.write('data: 输出结束\n\n')
            res.end()
        }
    }, 10)

    // 当客户端关闭连接时会触发
    req.on('close', () => {
        console.log('客户端关闭sse连接')
    })
})

// 启动服务
const PORT = process.env.PORT || 3000

app.listen(PORT, () => {
    console.log(`Server listening on port ${PORT}`)
})
```

##### 效果预览

![image-20241227161147850](/Users/admin/Library/Application Support/typora-user-images/image-20241227161147850.png)

#### 注意事项

1. 注意结尾\n 和\n\n的区别，\n代表换行，\n\n代表一条消息（chunk）的结束
2. 接口服务输出data: 时后面必须要加空格，否则输出后会出现首字符空格丢失的情况，示例 res.write(`data: ${chunk}\n\n`)

#### 封装

SSE不支持POST请求，我们可以基于 [开源库](https://github.com/Azure/fetch-event-source) 或者二次封装来满足业务自定义需求。 



#### 	