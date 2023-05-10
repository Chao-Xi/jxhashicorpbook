# **4 私密信息管理利器 HashiCorp Vault——验证和授权**

到目前为止，我们都是使用 vault 客户端直接访问服务器，并未进行任何登录之类的操作。这是因为在开发模式下，服务器会自动将用户登录为 root 用户，目的是为了简化测试，避免在登录问题上卡住初学者。但在生产环境中这显然是非常不安全的。**再重复一次，绝对不要在生产环境中使用开发模式。**

## 1 验证方法（Auth Method）

在用户认证的问题上，Vault 同样使用了灵活的插件架构，允许多种认证手段，Vault 将其称为验证方法（Auth method）

* AppRole
* AWS
* Google Cloud
* Kubernetes
* Github
* LDAP
* Okta
* RADIUS
* TLS Certificates
* Tokens

这些方法中，Token 是 Vault 内置的验证方法，启动时即被加载，且不能禁用。回忆一下，服务器启动时会输出 Root Token，使用 Root token 登录的用户具有系统最高的访问权限。Token 是可继承的，这个继承包括两方面的含义：

* **持有 Token 的用户创建新的 Token（Child token），除非特别指定，否则 Child token 权限和原来的 Token 相同；**
* **当回收（Revoke）某个 Token 时，其所创建的 Child Token，以及 Child Token 的 Child Token... 都会被一并删除**。

## 2 Token 验证方法

创建新一个新 Token：

```
$ vault token create
Key                Value
---                -----
token              0fea4b53-9e31-cd50-c5ae-104f66192297
token_accessor     14ec0d45-cfed-6a84-8498-923194e361df
token_duration     ∞
token_renewable    false
token_policies     [root]
```

当 Token 不再使用后，可以用 revoke 撤销：

```
$ vault token revoke 0fea4b53-9e31-cd50-c5ae-104f66192297
Success! Revoked token (if it existed)
```
重复第一步的操作再创建一个 Token。然后用该 Token 登录：

```
$ vault login bfdd4016-651a-91d1-b24f-7d032e9127fe
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                Value
---                -----
token              bfdd4016-651a-91d1-b24f-7d032e9127fe
token_accessor     e435c4ed-0662-3298-9644-7554fe4a2729
token_duration     ∞
token_renewable    false
token_policies     [root]
```

注意，虽然 Token 方法非常简便易用，但主要目的是为了支持自身的运行，安全性并不是非常高。

<mark>对于真正的用户/机器认证场景，Vault 官方推荐使用其他更加成熟的机制，例如 LDAP，Github，AppRole 等，并且这些方法通常有更好的工具支持</mark>。

## 3 其他 Auth Method

说明：在 Vault 的官方文档中，使用 Github 作为 Auth Method 的例子。遗憾的是，Github 的用户验证必须使用组织账号，而我只有个人账户，所以这里无法演示。有 Github 组织账户的同学可以按照 官方文档 中的步骤来操作。这里，我们用另一个简单而常用的 Auth Method 来演示，这就是 userpass。顾名思义，userpass 需要用户名/密码对来进行认证，这和大多数网站的用户管理机制是一致的。

为了使用 userpass 验证方法，首先需要启用它：

```
$ vault auth enable -path=userpass userpass
Success! Enabled userpass auth method at: userpass/
```
和其他的 Vault 对象类似，你可以用内置的 path-help 来查看该方法的帮助信息：

```
$ vault path-help auth/userpass
```
然后，添加用户/密码对：

```
$ vault write auth/userpass/users/bob password=male policies=admin
Success! Data written to: auth/userpass/users/bob

$ vault write auth/userpass/users/alice password=female policies=admin
Success! Data written to: auth/userpass/users/alice
```
Policies 参数和用户权限有关，后面我们再讲。现在你可以用用户名/密码登录了：

```
$ vault login -method=userpass username=bob password=male
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                    Value
---                    -----
token                  9e14802b-549b-d0b1-3212-b8b7fbf33f73
token_accessor         2f411ac4-22d1-7761-8c38-9f668e8c411e
token_duration         768h
token_renewable        true
token_policies         [admin default]
token_meta_username    bob
```

## 4 授权

上面的操作讲述的都是用户认证（authentication）的内容，其中也涉及到一些授权（authorization）的参数，但并未深入讲解。现在我们就来介绍关于授权的内容

为了实现授权，需要分两个步骤：

1. 指定授权策略（Policy）。你可以把 Vault Policy 理解为一般系统中常见的角色（Role），它定义为一些操作权限的集合。
2. 将策略（Policy）和特定的登录身份信息关联起来。

策略是用 HCL 语言编写的。[HCL](https://github.com/hashicorp/hcl) 是 HashiCorp 创造的、专门用于配置文件的语言格式，类似于 Json/YAML（为什么不用现成的 Json/YAML 呢？上面的链接里有来自官方的解释。）

HCL 的格式并不复杂。下面我们来编写一个简单的策略：

```
path "secret/*" {
  capabilities = ["create"]
}

path "secret/foo" {
  capabilities = ["read"]
}
```

规则很简单：我们希望对 secret/ 路径下的一般数据有完全的读写权限，但 secret/foo 则是只读的。

<mark>同时可以看出，HCL 的作用域规则类似于 CSS，后面的规则具有更高的优先级。</mark>

如果我们担心规则有语法错误，可以用 fmt 命令检查：

```
$ vault policy fmt my-policy.hcl 
Success! Formatted policy: my-policy.hcl
```
fmt 不仅会检查文件中有无语法错误，还会把文件格式整理成标准的形式。

接下来，我们将文件导入系统，生成一个对应的策略（Policy）。如果这一步发生 403 错误，请检查你是否是以 Root Token 登录：

```
$ vault policy write my-policy my-policy.hcl
Success! Uploaded policy: my-policy
```
我们根据该权限新建一个登录 Token：

```
vault token create -policy=my-policy
Key                Value
---                -----
token              26a6addd-485f-3203-9b2a-a426d3c91030
token_accessor     26006593-9106-8dcd-607b-67646b5f5052
token_duration     768h
token_renewable    true
token_policies     [default my-policy]
```
可以看到新 Token 的权限已经正确设置了。

用新的 Token 登录，并验证权限是否生效：

```
$ vault login 26a6addd-485f-3203-9b2a-a426d3c91030
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                Value
---                -----
token              26a6addd-485f-3203-9b2a-a426d3c91030
token_accessor     26006593-9106-8dcd-607b-67646b5f5052
token_duration     767h58m50s
token_renewable    true
token_policies     [default my-policy]
$ vault write secret/hello name=world
Success! Data written to: secret/hello
$ vault write secret/foo name=world
Error writing data to secret/foo: Error making API request.

URL: PUT http://127.0.0.1:8200/v1/secret/foo
Code: 403. Errors:

* permission denied
```
最后，我们还可以把新的策略和用户验证方法关联起来：

```
$ vault write auth/userpass/users/bob password=male policies=my-policy
Success! Data written to: auth/userpass/users/bob
$ vault login -method=userpass username=bob password=male
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                    Value
---                    -----
token                  66c0cd16-4dea-5180-a52f-a0b19e2f8341
token_accessor         88690d6d-fb5f-e3aa-4d5f-50ecbf78ae2b
token_duration         768h
token_renewable        true
token_policies         [default my-policy]
token_meta_username    bob
```

我们看到新的策略确实应用到用户了。

