# RPC客户端

我们实现一个简单的客户端。它适用于演示所有服务器模型。它的过程就是向服务器连续发送 10 个 RPC 请求，并输出服务器的响应结果。

## 消息协议

我们使用长度前缀法来确定消息边界，消息体使用 json 序列化。

每个消息都有相应的名称，请求的名称使用 in 字段表示，请求的参数使用 params 字段表示，响应的名称是 out 字段表示，响应的结果用 result 字段表示。

我们将请求和响应使用 json 序列化成字符串作为消息体，然后通过 Python 内置的 struct 包将消息体的长度整数转成 4 个字节的长度前缀字符串。

```json
// 输入
{
    in: "ping",
    params: "ireader 0"
}

// 输出
{
    out: "pong",
    result: "ireader 0"
}
```

## 客户端代码

```py
# rpc_client.py
import json
import time
import struct
import socket


# 请求长度前缀和响应长度前缀都是4字节长度
def rpc(sock, in_, params):
    request = json.dumps({'in': in_, 'params': params})  # 请求消息体
    length_prefix = struct.pack('I', len(request))       # 请求长度前缀

    sock.sendall(length_prefix)
    sock.sendall(request.encode())

    length_prefix = sock.recv(4)                         # 响应长度前缀
    length, = struct.unpack("I", length_prefix)          # 真正的响应长度
    body = sock.recv(length)
    response = json.loads(body)

    return response['out'], response['result']


if __name__ == "__main__":
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(('localhost', 8080))

    # 连续发送10个rpc请求
    for i in range(10):
        out, result = rpc(s, 'ping', f'ireader {i}')
        print(f'out = {out}, result = {result}')
        time.sleep(1)    # 休眠一秒，方便观察

    s.close()

```


