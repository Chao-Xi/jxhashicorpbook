# **6 私密信息管理利器 HashiCorp Vault——REST API**

我们一直在使用 Vault 命令行客户端。不过，部分输出内容也透露了这样的信息，那就是客户端和服务器的通信实质上是通过 HTTP 协议进行的。

本文就显示如何使用 REST API 和服务器通信。Vault 有许多针对特定语言的客户端库，它们基本上就是这些接口的简单封装。只要明白了基本原理，其实你自己写一个也非常简单。

我们已经知道，Vault 有许多用户认证方法，对于程序客户端，Token 是最简单也是最常用的方法。为了避免在后续操作中反复输入 Token，我先把它保存到环境变量中：

```
export VAULT_TOKEN=444b8cb7-e58d-9e57-076a-fe0763f06248
```
为简单起见，我在测试环境中直接使用 Root Token。生产环境下应该分配并使用权限受限的 Token。

```
$ curl \
> --header "X-Vault-Token: $VAULT_TOKEN" \
> --request POST \
> --data '{"bar": "baz", "name": "world"}' \
> http://127.0.0.1:8200/v1/secret/foo
```

写入成功并不会返回任何信息，但是你可以验证数据确实写入了：

```
$ curl \
> --header "X-Vault-Token: $VAULT_TOKEN" \
> http://127.0.0.1:8200/v1/secret/foo
{"request_id":"50ccf7ef-8153-ec3d-cfc6-ce52dc60483c","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"bar":"baz","name":"world"},"wrap_info":null,"warnings":null,"auth":null}
```
如果调用失败，将会返回错误信息：

```
curl \
> --header "X-Vault-Token: $VAULT_TOKEN" \
> --request POST \
> --data '{"bar": "baz", "name": "world"}' \
> http://127.0.0.1:8200/v1/mydata/foo
{"errors":["no handler for route 'mydata/foo'"]}
```

虽然可以使用的接口很丰富，但我们日常会用到的基本上也就是读写数据了。除非你需要做一个功能完善的工具，那么你可以参考完整的 [接口文档](https://www.vaultproject.io/docs/index.html)。
