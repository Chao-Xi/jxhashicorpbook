# **L3 Working with Tokens**

## Many Users & Systems - All Using Tokens

![Alt Image Text](../images/vt4_3_1.png "Body image")


**All users, applications, and systems rely on tokens to create and obtain secrets from Vault.** 

First, they get the tokens and then use them to gain access to target systems at the proper authorization levels.

**External users and systems also rely on Vault for authentication and authorization.**

These include cloud users, applications, and services, as well as mobile apps and remote employees. 

## Token Types

### Service Token

* **Regular**

	* ttl `and/or` 
	* max-ttl 
	* stored

Regular tokens with time to live that can also have a max time to live, which are stored internally by Vault. 

Regular tokens with time to live that can also have a max time to live, which are stored internally by Vault.

* **Use Limit**

	* `ttl & max-ttl` 
	* `# of uses` 
	* Expire @ maxttl or max # use

Use limit tokens that also have time to live and max time to live, but also add a number of times the token can be used into the mix.

These tokens expire either when the **max number of uses is reached or when their ttl period expires**. 

* **Periodic**

	* ttl
	* No max-ttl
	* Long-running services

Periodic tokens can only be created by root and sudo users. 

They have a ttl and no max‑ttl, and thus can be renewed indefinitely before they expire

**This token type is suitable for long‑running services whose ability to regenerate a token is problematic.** 

* **Short-lived**

	* Short ttl 
	* No max-ttl

Short‑lived tokens simply have a very short time to live, for example, 60 seconds, and no max time to live.

* **Orphan**
	* Do NOT expire when parent expires

These can be of any of the other service subtypes, but get revoked the moment their parent is revoked, **either when its tto expires or a Vault admin manually revokes their parent**.

* **Root**
	* Do NOT expire

	
### Batch

* Not stored - in-memory only 
* Light weight - limited features 
* Fixed ttl
* No accessor 
* Cannot be renewed 
* Must be expressly created: `-type=batch`
* Cannot be root tokens (the latter never expire)
* ID begins with “b.” 
* Can be used across Vault clusters 
* Designed for scalability

> Batch tokens are not persisted in backend storage. they're only kept in‑memory while in use. 
> 
> They're lightweight, which means they have limited features, like a fixed time to live and no accessor.
> 
> They cannot be renewed and must be expressly created with the `‑type=batch` option
> 
> Their ID begins with a "b," followed by a dot. 
> 
>  Can be used across Vault clusters with performance replication enabled. This feature is only available in Vault Enterprise and allows for batch tokens to be encrypted and sent across the barrier to other clusters.

Finally, they are designed for scalability, or more specifically, for cases where your application needs to scale quickly to hundreds or thousands of instances. 

![Alt Image Text](../images/vt4_3_3.png "Body image")


**Performance Replication - Vault Enterprise**

![Alt Image Text](../images/vt4_3_2.png "Body image")


## Demo: Token Types and Token Lifecycle

### Token Lease

![Alt Image Text](../images/vt4_3_4.png "Body image")

Leases don't live forever. 

They have a **time to live, or a TTL, which determines how long a token has access to secrets**. 

When the lease TTL is up, Vault expires the token by revoking it. 

Administrators can also revoke tokens manually using the CLI or API. 

Tokens can also be issued with a limited number of uses. 

**Leases force clients to periodically authenticate to Vault to maintain access to secrets.**

This prevents long‑lived secrets from being used to compromise data security. 


For token lifecycle management, just refer to this slide that **illustrates TTL, explicit Max TTL, and token renewal**


### **Leases**

* Tokens can be issued with limited number of uses
* Leases require clients to re-authenticate

![Alt Image Text](../images/vt4_3_5.png "Body image")




* **`devteam_policy.hcl`**

```
# Dev Team policy that allows members to do anythig with secrets related to this path
path "secret/data/devteam" {
   capabilities = [ "create", "read", "update", "delete" ]
}
```

```
# Create a policy for testing
vault policy write devteam devteam_policy.hcl

# Create a regular token without specifying a ttl or a max - we specify policy to avoid creating another root token
vault token create -policy=devteam_policy

$ vault token create -policy=devteam_policy
Key                  Value
---                  -----
token                hvs.CAESIH1u8Sj9UWGjhqCWXQy-JQyeLnYTxn6GmNCbdf4slRN4Gh4KHGh2cy5IVVJBbmoxdVNUT2RjQUhLaUxaank5Qnc
token_accessor       ZoRcoLZbklHz3uNItCynNybS
token_duration       768h
token_renewable      true
token_policies       ["default" "devteam_policy"]
identity_policies    []

$ vault token lookup hvs.CAESIH1u8Sj9UWGjhqCWXQy-JQyeLnYTxn6GmNCbdf4slRN4Gh4KHGh2cy5IVVJBbmoxdVNUT2RjQUhLaUxaank5Qnc
Key                 Value
---                 -----
accessor            ZoRcoLZbklHz3uNItCynNybS
creation_time       1671715446
creation_ttl        768h
display_name        token
entity_id           n/a
expire_time         2023-01-23T21:24:06.922434+08:00
explicit_max_ttl    0s
id                  hvs.CAESIH1u8Sj9UWGjhqCWXQy-JQyeLnYTxn6GmNCbdf4slRN4Gh4KHGh2cy5IVVJBbmoxdVNUT2RjQUhLaUxaank5Qnc
issue_time          2022-12-22T21:24:06.922439+08:00
meta                <nil>
num_uses            0
orphan              false
path                auth/token/create
policies            [default devteam_policy]
renewable           true
ttl                 767h59m38s
type                service

$ vault policy list | grep dev
devteam_policy
```

```
# Create regular token with ttl equal to 30 seconds and max ttl equalt to 60 seconds
vault token create -policy=devteam -ttl=30s -explicit-max-ttl=60s

$ vault token create -policy=devteam -ttl=30s -explicit-max-ttl=60s
Key                  Value
---                  -----
token                hvs.CAESIDBOJ6glyIN2JeCq4cdW8BrnHTtLVRc5HxsmT8oC2FbEGh4KHGh2cy5zMm9vbWRGMlkySWZacmh5S1NBQmR4ZDU
token_accessor       A30nfwE746aVSdhw8QtFW1In
token_duration       30s
token_renewable      true
token_policies       ["default" "devteam"]
identity_policies    []
policies             ["default" "devteam"]
```

```
 vault token create -policy=devteam -ttl=30s -explicit-max-ttl=60s
Key                  Value
---                  -----
token                hvs.CAESINGSFhX_Vc5wEnnFTw9qMb0V1BQrp46ukq572m6iqejSGh4KHGh2cy5lWXk4N2FhbkVLQnF3SUF1Nm5tSW5SS2c
token_accessor       IYghxZIunASGjp662CxNLPol
token_duration       30s
token_renewable      true
token_policies       ["default" "devteam"]
identity_policies    []
policies             ["default" "devteam"]


$ vault token lookup hvs.CAESINGSFhX_Vc5wEnnFTw9qMb0V1BQrp46ukq572m6iqejSGh4KHGh2cy5lWXk4N2FhbkVLQnF3SUF1Nm5tSW5SS2c
Key                 Value
---                 -----
accessor            IYghxZIunASGjp662CxNLPol
creation_time       1671715594
creation_ttl        30s
display_name        token
entity_id           n/a
expire_time         2022-12-22T21:27:04.477516+08:00
explicit_max_ttl    1m
id                  hvs.CAESINGSFhX_Vc5wEnnFTw9qMb0V1BQrp46ukq572m6iqejSGh4KHGh2cy5lWXk4N2FhbkVLQnF3SUF1Nm5tSW5SS2c
issue_time          2022-12-22T21:26:34.477522+08:00
meta                <nil>
num_uses            0
orphan              false
path                auth/token/create
policies            [default devteam]
renewable           true
ttl                 16s
type                service
```

### `vault token renew`

```
$  vault token renew hvs.CAESIDhAEwP-pMqGQpe5Z9tPHHQ5PLoUc8SwhZ6eFkkK244CGh4KHGh2cy51dTRaTWFiY1Byd1pUT05BdW91WlVTbGk
Key                  Value
---                  -----
token                hvs.CAESIDhAEwP-pMqGQpe5Z9tPHHQ5PLoUc8SwhZ6eFkkK244CGh4KHGh2cy51dTRaTWFiY1Byd1pUT05BdW91WlVTbGk
token_accessor       FWaIz6NW4zMCBLIydMbF1nIn
token_duration       30s
token_renewable      true
token_policies       ["default" "devteam"]
identity_policies    []
policies             ["default" "devteam"]
```

```
$  vault token lookup hvs.CAESIFhcTv6ykU4Kw7Zu016QKlsXsf3gBpVRrxI0_5A80ijRGh4KHGh2cy5qdWxncVk3N2xiVW5iU0UxR2NqVXVIcEg


....
last_renewal         2022-12-22T21:30:21.783806+08:00
last_renewal_time    1671715821
...
```


### Create Use Limit token

```
vault token create -policy=devteam -ttl=30s -explicit-max-ttl=60s -use-limit=3


vault token create -policy=devteam -ttl=2m -explicit-max-ttl=4m -use-limit=3
```

```
 vault token create -policy=devteam -ttl=30s -explicit-max-ttl=60s -use-limit=3
Key                  Value
---                  -----
token                hvs.CAESIAEL8l_QR6PkgBQKoTU7gjCvvG2wBWiklq4LgOMN1zckGh4KHGh2cy5PcG9GcjdqN2dFaUJBazNuajV0bng1NnE
token_accessor       RJQUfOWBZGlxdg58x0BE3Pj3
token_duration       30s
token_renewable      true
token_policies       ["default" "devteam"]
identity_policies    []
policies             ["default" "devteam"]

$ vault login hvs.CAESIAEL8l_QR6PkgBQKoTU7gjCvvG2wBWiklq4LgOMN1zckGh4KHGh2cy5PcG9GcjdqN2dFaUJBazNuajV0bng1NnE
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                hvs.CAESIAEL8l_QR6PkgBQKoTU7gjCvvG2wBWiklq4LgOMN1zckGh4KHGh2cy5PcG9GcjdqN2dFaUJBazNuajV0bng1NnE
token_accessor       RJQUfOWBZGlxdg58x0BE3Pj3
token_duration       20s
token_renewable      true
token_policies       ["default" "devteam"]
identity_policies    []
policies             ["default" "devteam"]
```

**Over 30s**

```
$ vault login hvs.CAESIAEL8l_QR6PkgBQKoTU7gjCvvG2wBWiklq4LgOMN1zckGh4KHGh2cy5PcG9GcjdqN2dFaUJBazNuajV0bng1NnE
Error authenticating: error looking up token: Error making API request.

URL: GET http://127.0.0.1:8200/v1/auth/token/lookup-self
Code: 403. Errors:

* permission denied
```

### Use limit 

```
$ vault login hvs.EiYGguIh98HdbF3NkfrzQhM6
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                hvs.EiYGguIh98HdbF3NkfrzQhM6
token_accessor       TrsQs8oybr7vyHRuJOJGGohH
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```


```
vault token create -policy=devteam -ttl=2m -explicit-max-ttl=4m -use-limit=3

$ vault token create -policy=devteam -ttl=2m -explicit-max-ttl=4m -use-limit=3
Key                  Value
---                  -----
token                hvs.CAESIG_fDU4XR2uronpEiwVkHWMWO8GqR0ubRnLQnMoTUaH1Gh4KHGh2cy5rUm9FNG9TOVJQMGRMcW40SlJ3RlV6SlA
token_accessor       EmVPTaGsZzJINQzVQL1N16Fj
token_duration       2m
token_renewable      true
token_policies       ["default" "devteam"]
identity_policies    []
policies             ["default" "devteam"]
```

```
vault login hvs.CAESIG_fDU4XR2uronpEiwVkHWMWO8GqR0ubRnLQnMoTUaH1Gh4KHGh2cy5rUm9FNG9TOVJQMGRMcW40SlJ3RlV6SlA
```

```
$ vault token lookup hvs.CAESIB3VTtkE7T_Xf5ALfsmzM6QHrWfoZkUQgqzwtbe2R0PsGh4KHGh2cy5scmEwSFduTExiVTdrWXRvajlmRzJjc2s
Key                 Value
---                 -----
...
num_uses            3
...
```

### Enable userpass authentication method

```
# Login as root
# Enable userpass authentication method

vault login  hvs.EiYGguIh98HdbF3NkfrzQhM6

vault auth enable userpass

# Create a sample user and log in with that user
vault write auth/userpass/users/tokentester password=tokentester policies=devteam

vault login -method=userpass username=tokentester password=tokentester

Key                    Value
---                    -----
token                  hvs.CAESIIr8pythMlIOWgdOFdN46bTQxtiCYdE2ZLaxfiJOF7xbGh4KHGh2cy5OVUtBc2V6TWZrSVI0UFF4ZkhLQlZzOGs
token_accessor         VdnFoTgvOUT78lSVXeiSqPYb
token_duration         768h
token_renewable        true
token_policies         ["default" "devteam"]
identity_policies      []
policies               ["default" "devteam"]
token_meta_username    tokentester


# Attempt to create periodic token
vault token create -policy=devteam -period=6h # Fails because we are not root or sudo user

Error creating token: Error making API request.

URL: POST http://127.0.0.1:8200/v1/auth/token/create
Code: 403. Errors:

* 1 error occurred:
        * permission denied

# Try with root

$ vault login hvs.EiYGguIh98HdbF3NkfrzQhM6

$ vault token create -policy=devteam -period=6h # OK
Key                  Value
---                  -----
token                hvs.CAESIHuf_jIdABA6VM78zqw9ReS51Kw1iy6LEl4GD5NTxqOOGh4KHGh2cy4wVUcxZEZCTTNzaDFRRkZrQVQxQzJmbDk
token_accessor       6KSmLWarfZTEmClANRjnG37G
token_duration       6h
token_renewable      true
token_policies       ["default" "devteam"]
identity_policies    []
policies             ["default" "devteam"]

$ vault token lookup hvs.CAESIHuf_jIdABA6VM78zqw9ReS51Kw1iy6LEl4GD5NTxqOOGh4KHGh2cy4wVUcxZEZCTTNzaDFRRkZrQVQxQzJmbDk

$ vault token lookup hvs.CAESIHuf_jIdABA6VM78zqw9ReS51Kw1iy6LEl4GD5NTxqOOGh4KHGh2cy4wVUcxZEZCTTNzaDFRRkZrQVQxQzJmbDk
Key                 Value
---                 -----
accessor            6KSmLWarfZTEmClANRjnG37G
creation_time       1671717703
creation_ttl        6h
display_name        token
entity_id           n/a
expire_time         2022-12-23T04:01:43.545968+08:00
explicit_max_ttl    0s
id                  hvs.CAESIHuf_jIdABA6VM78zqw9ReS51Kw1iy6LEl4GD5NTxqOOGh4KHGh2cy4wVUcxZEZCTTNzaDFRRkZrQVQxQzJmbDk
issue_time          2022-12-22T22:01:43.545971+08:00
meta                <nil>
num_uses            0
orphan              false
path                auth/token/create
period              6h
policies            [default devteam]
renewable           true
ttl                 5h58m5s
type                service
```

### Periodic Tokens

* **Often No Max TTL**

Long one when they do week, month, etc.

* **Fit for Systems / Apps**

Not Humans - password reset for them is different

* **Repeated Actions**

Every 5 min, every week, every month, etc.

* **Batch Jobs**

Payroll, reconciliation, processing returns, etc.

* **Frequent Events**

Generating an insurance quote

In some cases, having a token be revoked would be problematic -- for instance, **if a long-running service needs to maintain its SQL connection pool over a long period of time. In this scenario, a periodic token can be used**. 

Periodic tokens can be created in a few ways:

* By having
sudo capability or a
root token with the `auth/token/create` endpoint
* By using token store roles
* **By using an auth method that supports issuing these, such as AppRole**



### Token role


Please note that when we use **token store role**, the current **value of the role's period setting will be used when the token is being renewed**, 

Which implies that **if you change the value by updating the role, that new value will be used next time the token is renewed**. 

**Max TTL** specified - **Vault will revoke token after the explicit max TTL expires**

![Alt Image Text](../images/vt4_3_6.png "Body image")

![Alt Image Text](../images/vt4_3_7.png "Body image")

```
# Create short-lived token
vault token create -policy=devteam -ttl=15s


vault login -method=userpass username=tokentester password=tokentester

$ vault login -method=userpass username=tokentester password=tokentester
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                    Value
---                    -----
token                  hvs.CAESINPv9wsw2b73HrLJYBagYgcXN5hJCR-N2ReFCdAQy_xfGh4KHGh2cy5FRzVOcmcwb2NVQURURGRWclhHaG5TdXk
token_accessor         wFij2qnXtytj2ze8z4LZElum
token_duration         768h
token_renewable        true
token_policies         ["default" "devteam"]
identity_policies      []
policies               ["default" "devteam"]
token_meta_username    tokentester


$ vault token create -orphan
Error creating token: Error making API request.

URL: POST http://127.0.0.1:8200/v1/auth/token/create
Code: 403. Errors:

* 1 error occurred:
        * permission denied
```

After logging in with token tester, we attempt to create an orphan token, but we fail because our user does not have sudo privileges.

```
vault login hvs.EiYGguIh98HdbF3NkfrzQhM6

vault token create -orphan

$ vault token create -orphan
Key                  Value
---                  -----
token                hvs.2V4XRoX99MV520WXmbIvKLiI
token_accessor       TA23xPGQhuLnbpplszEt5xqM
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```

### **Root Tokens Best Practices**

* Avoid creating additional root tokens
* Revoke root token(s) after initial configuration

A parent token prevents the child tokens under it to be renewed indefinitely. When the parent token expires or gets manually revoked, all child tokens it spawned are also revoked

By contrast, **orphan tokens continue to exist independently of the token that was used to create them**. 

### **Token Hierarchies**


**When parent TTL expires or parent is revoked, children also expire or are revoked**

### Create orphan token

```
# Crate a batch token
vault token create -type=batch -policy=devteam -ttl=30m
```

## Response Wrapping

### Response Wrapping

* **Level of indirection** - retrieve hidden secret， or a **hidden token**
* Secret / token retrieval happens only once


### **Single Retrieval**

* **Single client can retrieve**

Reduced risk of secret exposure

* **Trigger an alert**

In case of bad-actor intercept

> In case of bad actor intercepts the wrapping token and attempts to use it to get to the secret after our client has already used the wrapping token.

### Response Wrapping

Response wrapping tends to be used in situations where you lack a reliable authentication mechanism like AWS Identity and Access Management or Azure's Active Directory.  
 
* Lack of reliable authentication method 
* **Additional layer of security - guaranteethe secret ends up with the target recipient**
 

### Tokens as Secrets
 
As you can probably already suspect, t**okens themselves become secrets and need to be protected**. This is especially true for keys delivered via automation to applications and systems.


There is a special term HashiCorp uses for t**he secure delivery of keys to deployed applications. It is called Secure Introduction**. 


**Secrets Delivered by Automation Systems**

![Alt Image Text](../images/vt4_3_8.png "Body image")


### Automation System Categories

The keys are usually delivered by an automation system. The categories of such systems include: 

* Container orchestrators, the main ones being Kubernetes and Docker Swarm, 
* Continuous integration and continuous delivery systems like Jenkins, 
* Configuration management systems like Chef, Puppet, and Ansible. 


These systems need to be configured to connect to Vault, then generate a token from Vault, and embed it in a software build or another construct with a high degree of automation. 

![Alt Image Text](../images/vt4_3_9.png "Body image")


### Response Wrapping Cases


* **Wrap exiting secret**

Written to a path in Vault

* **Secret does not yet exit**

Even Vault admins cannot see it


**Then the applications that need the secrets can request them directly from Vault using the embedded wrapping token.** 

There are two cases that we can apply response wrapping to. 


* The first one involves wrapping a secret that already exists and was written to a path in Vault. 
* And the second case is when we want to generate a token or a secret that does not yet exist, **and we don't want even the Vault administrator to be able to see it, unless of course, the administrator decides to go ahead and use the wrapping token that will also be generated.** 



### Wrapping Token Use Restrictions

For the second case, the wrapping token is limited to only two uses, **once to write the wrapped token, and once to read it**.

* **Write wrapped token**

This happens when the new secret is generated

* **Read wrapped token**

**This happens when a client uses the wrapping token to get to the secret**

**<mark>This wrapped or permanent token is stored in a cubbyhole of the cubbyhole secrets engine. </mark>**


```
# These are the general commands
vault token renew [<token_id> | <accessor_id>]
vault token revoke [<token_id> | <accessor_id>]
```

```

# Create secret and then wrap existing secret
vault kv put secret/db-admin pwd=dbadmin_pwd

$ vault kv put secret/db-admin pwd=dbadmin_pwd
==== Secret Path ====
secret/data/db-admin

======= Metadata =======
Key                Value
---                -----
created_time       2022-12-23T15:19:31.896734Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            1

vault kv get -wrap-ttl=420 secret/db-admin

$ vault kv get -wrap-ttl=420 secret/db-admin
Key                              Value
---                              -----
wrapping_token:                  hvs.CAESILdV_ojWejCHAOreO4P-I5SnQlVi5bBafTs97yjGvnbUGh4KHGh2cy5nNlBXWkd1bkQyVUxObktCcThUSWhLU08
wrapping_accessor:               TdNcOIZiiwH6MbZSnrBTnNaw
wrapping_token_ttl:              7m
wrapping_token_creation_time:    2022-12-23 23:21:20.044418 +0800 CST
wrapping_token_creation_path:    secret/data/db-admin
```

```
curl --header "X-Vault-Token: hvs.CAESILdV_ojWejCHAOreO4P-I5SnQlVi5bBafTs97yjGvnbUGh4KHGh2cy5nNlBXWkd1bkQyVUxObktCcThUSWhLU08" --request POST $VAULT_ADDR/v1/sys/wrapping/unwrap | jq


 curl --header "X-Vault-Token: hvs.CAESILdV_ojWejCHAOreO4P-I5SnQlVi5bBafTs97yjGvnbUGh4KHGh2cy5nNlBXWkd1bkQyVUxObktCcThUSWhLU08" --request POST $VAULT_ADDR/v1/sys/wrapping/unwrap | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   317  100   317    0     0  23165      0 --:--:-- --:--:-- --:--:--  103k
{
  "request_id": "3f1e3232-081b-e056-3bf1-ca85fb48765f",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": {
    "data": {
      "pwd": "dbadmin_pwd"
    },
    "metadata": {
      "created_time": "2022-12-23T15:19:31.896734Z",
      "custom_metadata": null,
      "deletion_time": "",
      "destroyed": false,
      "version": 1
    }
  },
  "wrap_info": null,
  "warnings": null,
  "auth": null
}
```

```
# Generate and store a token as a secret in a cubbyhole 
vault token create -wrap-ttl=420 -policy=devteam

$ vault token create -wrap-ttl=420 -policy=devteam
Key                              Value
---                              -----
wrapping_token:                  hvs.CAESIB6OZIdLlNUsLV7NrheNG9Q9zZxnH8vJb5Nc_OdovLb8Gh4KHGh2cy5PSFZaNVVIeGMyTnhqYUt1bzRSWFk3WVE
wrapping_accessor:               BdCRU6wdKyLlyAMpQBvlTtdO
wrapping_token_ttl:              7m
wrapping_token_creation_time:    2022-12-23 23:23:32.440229 +0800 CST
wrapping_token_creation_path:    auth/token/create
wrapped_accessor:                AFcuSny3EAbOvw1GPpXRhOya


vault unwrap hvs.CAESIB6OZIdLlNUsLV7NrheNG9Q9zZxnH8vJb5Nc_OdovLb8Gh4KHGh2cy5PSFZaNVVIeGMyTnhqYUt1bzRSWFk3WVE

$ vault unwrap hvs.CAESIB6OZIdLlNUsLV7NrheNG9Q9zZxnH8vJb5Nc_OdovLb8Gh4KHGh2cy5PSFZaNVVIeGMyTnhqYUt1bzRSWFk3WVE
Key                  Value
---                  -----
token                hvs.CAESIJke72mTeSOK1BtSqsUUbT6H_0I2omptwS_4V_slfQqgGh4KHGh2cy45MTF5WHowMElxbDVncHpsMDlaNzZyZko
token_accessor       AFcuSny3EAbOvw1GPpXRhOya
token_duration       768h
token_renewable      true
token_policies       ["default" "devteam"]
identity_policies    []
policies             ["default" "devteam"]
```

## LDAP Authentication Method

 We're going to use Vault in combination with Apache Directory Studio, which is a completely free, open‑source implementation of the LDAP protocol. 
 
 
First, **enable the LDAP authentication method**. Then, **we need a policy for our dev team**, so let's upload one. The policy itself just gives us the run‑of‑the‑mill CRUD capabilities on that path. 

```
 # Enable and configure LDAP authentication method; cn=vaultadmin sn=vaultadmin userPassword=vaultadmin
# billykid, pwd=b1g23f0ot, same for gcharm user
# groupdn="ou=groups,ou=system" \
# groupfilter="(&(objectClass=person)(cn={{.Username}}))" \
```


* Configuring Vault to talk to our open source LDAP server. 
* We are writing to the `auth/ldap/config` path, and as part of the configuration, we specify the URL that begins with the LDAP protocol, followed by the domain of our server and its port number, 10389. 
* We set our **username** to cn, or common name, that is essentially equivalent to user id in our case. 
* This is followed by the user distinguished name, the common identifier for entities in LDAP where the first component, cn, is set to vaultadmin. 
* `groupdn` that will be used to find out what group our user belongs to. 
* `binddn` here is the same as the userdn, which specified the bindpass.
*  The value here needs to match what we have set up in LDAP.
* Groupattr is set to uniqueMember
* We set `insecure_tls` to true because for testing purposes, we don't need any encryption, and correspondingly, we specify false for starttls.


```
vault auth enable ldap
vault write auth/ldap/config \
url="ldap://example.com:10389" \
userattr="cn" \
userdn="cn=vaultadmin,ou=users,ou=system" \
groupdn="ou=groups,ou=system" \
groupfilter="(|(memberUid={{.Username}})(member={{.UserDN}})(uniqueMember={{.UserDN}}))"
binddn="cn=vaultadmin,ou=users,ou=system" \
bindpass="vaultadmin" \
groupattr="uniqueMember" \
insecure_tls=true \
starttls=false
```

PRODUCTION

* `insecure tls=false`
* `starttls=true`


For production, you will have to set the last two options to false and true respectively and specify the CA certificate with the certificate option whose value is a file name preceded by an at sign

```
# Valid search base:
# cn=vaultadmin,ou=users,ou=system
# filter="(objectClass=*)" \ - to filter users but is not part of the Vault documentation
```

**`devteam_policy.hcl`**

```
# Dev Team policy that allows members to do anythig with secrets related to this path
path "secret/data/devteam" {
   capabilities = [ "create", "read", "update", "delete" ]
}
```

```
# Upload the policy
vault policy write devteam devteam_policy.hcl
```

```
# Create a user group in vault to match the setup on the LDAP server
vault write auth/ldap/groups/devstars policies=devteam
```

```
# No need to create a user in vault - vaultadmin with password "vaultadmin" was already created on the LDAP server
vault login -method=ldap username=vaultadmin
# vaultadmin

# Log in with LDAP method
vault login -method=ldap username=vaultadmin
```

![Alt Image Text](../images/vt4_3_10.png "Body image")

![Alt Image Text](../images/vt4_3_11.png "Body image")

![Alt Image Text](../images/vt4_3_12.png "Body image")


```
# Sample search
# LDAP Search Example
ldapsearch -h localhost -p 10389 -x -b "dc=example,dc=com" 'cn=billykid'

# which returned
# extended LDIF
#
# LDAPv3
# base <dc=example,dc=com> with scope subtree
# filter: cn=billykid
# requesting: ALL
#

# search result - "my note: the next two lines did not have a comment #, I put it in."
# search: 2
# result: 0 Success

# numResponses: 1
```

![Alt Image Text](../images/vt4_3_13.png "Body image")


## TLS Certificates and PKI

**Using TLS certificates makes the connection between client and server.**

They're secure. 

The hard part about certificates is managing them. **They need to be generated, distributed, configured for use, revoked, and kept track of, and finally, be renewed or reissued**. 

### TLS Certificate Management

* **Generate**

Via OpenSSL or commercial tools

* **Distributed**

You need to send them securely to many clients

* **Configured for use**

Browsers and other clients need to be set up

* **CRLs Management**

Certificate revocation lists need to be maintained

* **Renew or Reissue**

Not easy to do on a mass scale. People often forget

![Alt Image Text](../images/vt4_3_14.png "Body image")

**TLS Certificate Chain**

![Alt Image Text](../images/vt4_3_15.png "Body image")

### Letsencrypt

* Free TLS certificates with DNS zone
* DNS zone is is a distinct part of the domain namespace managed by a legal entity

**`selfsigned.cfr`**

```
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no

[req_distinguished_name]
C = CA
ST = BC
L =  Vancouver
O = Globomantics
CN = *

[v3_req]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
basicConstraints = CA:TRUE
subjectAltName = @alt_names

[alt_names]
DNS.1 = *
DNS.2 = *.*
DNS.3 = *.*.*
DNS.4 = *.*.*.*
DNS.5 = *.*.*.*.*
DNS.6 = *.*.*.*.*.*
DNS.7 = *.*.*.*.*.*.*
IP.1 = 127.0.0.1
```

```
$ openssl req -x509 -batch -nodes -newkey rsa:2048 -keyout selfsigned.key -out selfsigned.crt -config selfsigned.cfr -days 365

Generating a RSA private key

writing new private key to
'selfsigned.key

$ ll
cert.pem
key.pem
selfsigned.cfr
selfsigned.crt
selfsigned.key
```

```
sudo cp -p selfsigned.* /opt/vault/tls

sudo chown vault:vault /opt/vault/tls/selfsigned.crt

sudo chown vault:vault /opt/vault/tls/selfsigned.kev

sudo ls -1 /opt/vault/tis
```

**`vault.hcl`**

```
# Full configuration options can be found at https://www.vaultproject.io/docs/configuration

ui = true

#mlock = true
#disable_mlock = true

storage "file" {
  path = "/opt/vault/data"
}

#storage "consul" {
#  address = "127.0.0.1:8500"
#  path    = "vault"
#}

# HTTP listener
#listener "tcp" {
#  address = "127.0.0.1:8200"
#  tls_disable = 1
#}

# HTTPS listener
listener "tcp" {
  address       = "127.0.0.1:8200"
  tls_cert_file = "/opt/vault/tls/selfsigned.crt"
  tls_key_file  = "/opt/vault/tls/selfsigned.key"
}
```

```
sudo vault server -config=/etc/vault.d/vault.hel
```

![Alt Image Text](../images/vt4_3_16.png "Body image")

```
vault operator init

Unseal Key 1: GYeeavH0KXykZb7TgDYGlz8mTxbch6nGGcrUipLMiws]
Unseal Key 2: 1DHyX/RXg61paJ4wyfWxv08PU+9501dMJ70AA/puCP15
Unseal Key 3: 2uP6GJdQECq0L2fuRBTNx3q7A0PO+pdpqUXpmgeVq+jn
Unseal Key 4: usqDmFy6n(3MEDQwaj 7kGbXkfKYNq13WusVPsOlsSWVG
Unseal Key 5: wFrUAfFKFLiRO/dgRDt5gaTcYq3ckDXPB202qLIBH6SZ


Initial Root Token: s.HF5qwhddBJj1dvLU82Br1HDc
Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.
Vault does not store the generated master key. Without at least 3 keys to
reconstruct the master key, Vault will remain permanently sealed!
It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```

```
$ vault operator unseal GYeeavH0KXykZb7TgDYGlz8mTxbch6nGGcrUipLMiws]

$ vault operator unseal 1DHyX/RXg61paJ4wyfWxv08PU+9501dMJ70AA/puCP15

$ vault operator unseal 2uP6GJdQECq0L2fuRBTNx3q7A0PO+pdpqUXpmgeVq+jn
```

### Set up Root CA

```
###########################
#      Set up Root CA     #
###########################

# Enable cert secrets engine
vault secrets enable pki

# Bump up the max lease ttl to 10 years
vault secrets tune -max-lease-ttl=87600h pki

 Generate root cert
vault write -format=json pki/root/generate/internal \
common_name=”vault-ca-root-pki” | tee \
>(jq -r .data.certificate > vault-ca-root-pki.pem) \
>(jq -r .data.issuing_ca > vault-ca-root-pki-issuing.pem) \
>(jq -r .data.private_key > vault-ca-root-pki-key.pem)
```

![Alt Image Text](../images/vt4_3_17.png "Body image")

```
# Setup CRL and issuing URL
vault write pki/config/urls issuing_certificates="http://127.0.0.1:8300/v1/pki/ca" crl_distribution_points="http://127.0.0.1:8300/v1/pki/crl"

# Create intermediate CA
vault secrets enable -path=pki_int pki
vault secrets tune -max-lease-ttl=43800h pki_int

# Generate intermediate CA signed request
vault write -format=json pki_int/intermediate/generate/internal \
common_name="vault-ca-root-pki Intermediate Authority" \
| jq -r '.data.csr' > vault-ca-root-pki-int.csr

# Setting up our Intermediate CA requires signing our CSR using our Root CA
vault write -format=json pki/root/sign-intermediate csr=@vault-ca-root-pki-int.csr \
common_name="vault-ca-root-pki-int" \
format=pem_bundle ttl="43800h" \
| jq -r '.data.certificate' > vault-ca-root-pki-int.pem

#  Inject intermediate certificate into Intermediate CA config
vault write pki_int/intermediate/set-signed certificate=@vault-ca-root-pki-int.pem

# Configure URL
vault write pki_int/config/urls issuing_certificates="http://127.0.0.1:8300/v1/pki_int/ca" crl_distribution_points="http://127.0.0.1:8300/v1/pki_int/crl"
```

### Authenticate with Cert 

```
###########################
# Authenticate with Cert  #
###########################
# Enable cert auth method
vault auth enable cert


# Enable the PKI secrets engine at path "pki_int"
vault secrets enable -path=pki_int pki # Done previously because I executed the command as part of authentication below
# Create a role
vault write pki_int/roles/vault-cert allow_any_name=true max_ttl="1440h" generate_lease=true

# Upload the policy
vault policy write vault-cert vault-cert.hcl
```

* `vault-cert.hcl`

```
path "pki_int/issue/*" {
    capabilities = ["create", "update"]
}

path "pki_int/certs" {
    capabilities = ["list"]
}

path "pki_int/revoke" {
    capabilities = ["create", "update"]
}

path "pki_int/tidy" {
    capabilities = ["create", "update"]
}

path "pki/cert/ca" {
    capabilities = ["read"]
}

path "auth/token/renew" {
    capabilities = ["update"]
}

path "auth/token/renew-self" {
    capabilities = ["update"]
}

# Roles to create, update secrets
path "/sys/mounts" {
    capabilities = ["read", "update", "list"]
}

path "/sys/mounts/*" {
capabilities = ["update", "create"]
}

path "sys/policies/acl" {
    capabilities = ["read"]
}

path "secret/*" {
    capabilities = ["read", "create", "update", "delete"]
}
```

```
# Generate certificates
vault write -format=json pki_int/issue/vault-cert \
common_name="vault-cert" | tee \
>(jq -r .data.certificate > vault-cert-certificate.pem) \
>(jq -r .data.issuing_ca > vault-cert-issuing-ca.pem) \
>(jq -r .data.private_key > vault-cert-private-key.pem)

$ Upload the certificate to Vault and specify the policy for the users of that certificate
vault write auth/cert/certs/vault-cert \
display_name=vault-cert \
policies=vault-cert \
certificate=@vault-cert-certificate.pem

# Log in with our certs
vault login -method=cert -client-cert=vault-cert-certificate.pem -client-key=vault-cert-private-key.pem name=vault-cert 
```

![Alt Image Text](../images/vt4_3_18.png "Body image")

## Using the AppRole Authentication Method with Jenkins

### AppRole with Jenkins and Vault

![Alt Image Text](../images/vt4_3_19.png "Body image")

```
# Install the latest LTS version 
brew install jenkins-lts

# Get intiial admin password
sudo cat /Users/$(whoami)/.jenkins/secrets/initialAdminPassword

# Install a specific LTS version: 
brew install jenkins-lts@YOUR_VERSION

# Start/restart/stop/ the Jenkins service: 
brew services start jenkins-lts
brew services restart jenkins-lts
brew services stop jenkins-lts

# Upgrade the Jenkins version: 
brew upgrade jenkins-lts

sudo cat /Users/$(whoami)/.jenkins/secrets/initialAdminPassword


4d21978f240c41a5aa482323994a1162
```



![Alt Image Text](../images/vt4_3_20.png "Body image")

### Install Vault plugin and configure vault secret

![Alt Image Text](../images/vt4_3_21.png "Body image")

```
vault login root

# Create a secret
vault kv put secret/devops/jenkinssecret uid=dbadmin pwd=sup3rb@dsEcr1t
Success! Data written to: secret/devops/jenkinssecret

# Enable AppRole authentication method
vault auth enable approle
```

**Upload Jenkins policy to Vault**

```
vault policy write jenkins jenkins.hcl
Success! Uploaded policy: jenkins
```

**`jenkins.hcl`**

```
# Ability to login
path "auth/approle/login" {
    capabilities = ["create", "read", "list"]
}

# List and read devops secrets
path "secret/devops/*" {
    capabilities = ["read", "list"]
}
```

```
$ vault policy write jenkins jenkins.hcl
Success! Uploaded policy: jenkins

$ vault write auth/approle/role/jenkins policies=jenkins
Success! Data written to: auth/approle/role/jenkins

# Get the role-id a and generate a secret-id
$ vault read auth/approle/role/jenkins/role-id

$ vault write -f auth/approle/role/jenkins/secret-id
# role_id    97bab030-871b-31ba-1f7c-3b1036d7cc55
# secret_id             8805d584-c12e-9b34-4f33-bb596b968467
# secret_id_accessor    f75eef4e-fb88-f745-dafa-3ee5b671a87c
# secret_id_ttl         0s
# Description: Jenkins will use these credentials to authenticate to Vault
```

```
vault read auth/approle/role/jenkins/role-id
Key       Value
role_id   97bab030-871b-31ba-1f7c-3b1036d7cc55
```

### **Unlock Jenkins**

![Alt Image Text](../images/vt4_3_22.png "Body image")

![Alt Image Text](../images/vt4_3_23.png "Body image")

![Alt Image Text](../images/vt4_3_24.png "Body image")

### Create Jenkisn pipeline to test

`Test_ Jenkins_to_Vault`

![Alt Image Text](../images/vt4_3_25.png "Body image")

**Vault plugin**

* VAULT URL: http://127.0.0.1:8200
* Path： `secret/devops/jenkinssecret`
* Environment Variable: `VAULT_PROVIDED_SECRET`
* Key Name: uid
* K/V Engine Version : 1

![Alt Image Text](../images/vt4_3_26.png "Body image")

![Alt Image Text](../images/vt4_3_27.png "Body image")

## AWS Authentication Method

 With the AWS authentication method, we'll be relying on AWS entities, and I repeat, AWS entities to help with the authentication. These are users, groups, and so on, which have more or less permanent credentials. 
 
* These entities can assume roles, which serve as containers for a set of permissions grouped into policies that allow the entities to perform certain actions like reading AWS RDS instance metadata, starting and stopping EC2 instances, etc. 

### Authenticate and Play with Secrets
 
 ![Alt Image Text](../images/vt4_3_28.png "Body image")
 
**Entities**

* Users - have more or less permanent credentials
* Groups - users can become members of groups
* Roles - users can assume roles to perform actions

## Authentication Workflow

There are two authentication types present in the aws auth method: `iam` and `ec2`

With the `iam` method, a special AWS request signed with AWS IAM credentials is used for authentication. 

The IAM credentials are automatically supplied to AWS instances in AM instance profiles, Lambda functions, and others, and it is this information already provided by AWS which Vault can use to authenticate clients.

With the ec2 method **AWS is treated as a Trusted Third Party** and cryptographically signed dynamic metadata information that uniquely represents each EC2 instance is used for authentication. This metadata information is automatically supplied by AWS to all EC2 instances.

Based on how you attempt to authenticate, Vault will determine if you are attempting to use the `iam` or `ec2` type. Each has a different authentication workflow, and each can solve different use cases.

Note: The `ec2` method was implemented before the primitives to implement the `iam` method were supported by AWS. 

The `iam` method is the recommended approach as it is more flexible and aligns with best practices to perform access control and authentication. See the section on comparing the two auth methods below for more information.

 ![Alt Image Text](../images/vt4_3_29.png "Body image")

 ![Alt Image Text](../images/vt4_3_30.png "Body image")


### **AWS Authentication Method**

* Set up AWS credentials and Vault role and policy
* Create related AWS role and policy plus an IAM user
* Link the vault role to the AWS role or user
* Log in using AWS as a Trusted 3rd party

### Demo

* Set up AWS credentials and Vault role and policy
* Create related AWS role and policy plus an IAM user
* Link the vault role to the AWS role or user
* Log in using AWS as a Trusted 3rd party

```
$ vault secrets list
Path          Type         Accessor              Description
----          ----         --------              -----------
cubbyhole/    cubbyhole    cubbyhole_05f910b5    per-token private secret storage
identity/     identity     identity_a798004e     identity store
secret/       kv           kv_cca718d1           key/value secret storage
secretv1/     kv           kv_205e10b5           KVv1 for Policy Parameter Testing
sys/          system       system_c13790f5       system endpoints used for control, policy and debugging

vault secrets enable -path=secret kv
Success! Enabled the kv secrets engine at: secret/

# Write some test data
vault kv put secret/awsauthdemo/config ttl=10m username=devuser password=b3st4secret

# Enable AWS authentication method
vault auth enable aws
Success! Enabled aws auth method at: aws/

vault policy write awsauth awsauth.hcl
```

* `awsauth.hcl`

```
# Demo AWS authentication method policy
path "secret/awsauthdemo/*" {
    capabilities = ["read", "list"]
}
```

```

# Configure the AWS credentials
vault write auth/aws/config/client secret_key=N8F8L3CmthvxwHrGPjoALBjRe9khWFxrfbCma/y9 access_key=AKIATPLNME26BEGUOLUL
```

### Create a separate AM user and generate access and secret keys

**Vault looks for access and secret keys at the default locations, e.g.**  `~/.aws/credentials`

**Override AWS Key Set Values**

```
# Create AWS policy and role, then attach the policy to the role
aws iam create-policy --policy-name vault-docs-awsauth-policy --policy-document file://awsauth_recommended_policy.json
```

* `awsauth_recommended_policy.json`

```
{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "ec2:DescribeInstances",
          "iam:GetInstanceProfile",
          "iam:GetUser",
          "iam:GetRole"
        ],
        "Resource": "*"
      },
      {
        "Effect": "Allow",
        "Action": ["sts:AssumeRole"],
        "Resource": ["arn:aws:iam::239136941756:role/vault-aws-auth-role"]
      },
      {
        "Sid": "ManageOwnAccessKeys",
        "Effect": "Allow",
        "Action": [
          "iam:CreateAccessKey",
          "iam:DeleteAccessKey",
          "iam:GetAccessKeyLastUsed",
          "iam:GetUser",
          "iam:ListAccessKeys",
          "iam:UpdateAccessKey"
        ],
        "Resource": "arn:aws:iam::*:user/${aws:username}"
      }
    ]
}
```

```
aws iam create-role --role-name vault-aws-auth-role --assume-role-policy-document file://vault-aws-auth-role.json
```

* `vault-aws-auth-role.json`

```
{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "Service": "iam.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
      },
      {
        "Effect": "Allow",
        "Principal": {
          "AWS": "arn:aws:iam::239136941756:root"
        },
        "Action": "sts:AssumeRole"
      }
    ]
  }
```

### Attach policy to iam role

```
aws iam attach-role-policy --role-name vault-aws-auth-role --policy-arn arn:aws:iam::239136941756:policy/vault-docs-awsauth-policy
```
```
 Create the vault-aws-auth-role manually in AWS IAM console and attach to it the vault-docs-awsauth-policy just created above
# Or user the attach-role-policy iam command, as shown above
# To use this: delete prior versions: aws iam delete-policy --policy-arn arn:aws:iam::<aws_account_number>:policy/vault-docs-awsauth-policy
# Create a Vault role and associate it with an AWS role that has the custom policy we create previously
# bound_iam_principal_arn="arn:aws:iam::<aws_account_number>:role/vault-aws-auth-role" \
# /RoleSessionName
# bound_iam_principal_arn="arn:aws:iam::<aws_account_number>:user/*" \
# bound_iam_principal_arn="arn:aws:iam::<aws_account_number>:role/vault-aws-auth-role" \
# vault write auth/aws/role/awsauth-role-iam \
#     auth_type=iam \
#     bound_iam_principal_arn="arn:aws:iam::<aws_account_number>:role/vault-aws-auth-role" resolve_aws_unique_ids=true \
#     policies=awsauth ttl=24h
```
 ![Alt Image Text](../images/vt4_3_31.png "Body image")

 ![Alt Image Text](../images/vt4_3_32.png "Body image")

```
# Works OK, with IAM user vaultiam (created manually via IAM console)
vault write auth/aws/role/awsauth-role-iam \
    auth_type=iam \
    bound_iam_principal_arn="arn:aws:iam::239136941756:user/vaultiam" resolve_aws_unique_ids=true \
    policies=awsauth ttl=48h
```

```
# Allow any Principal to log in - tested OK
vault write auth/aws/role/awsauth-role-iam \
    auth_type=iam \
    bound_iam_principal_arn="arn:aws:iam::239136941756:*" resolve_aws_unique_ids=true \
    policies=awsauth ttl=48h
```

```
# Configure X-Vault-AWS-IAM-Server-ID Header - recommended
vault write auth/aws/config/client iam_server_id_header_value=vault.example.com
```

```
# Log in - IAM auth method
vault login -method=aws header_value=vault.example.com role=awsauth-role-iam
```

![Alt Image Text](../images/vt4_3_33.png "Body image")

* AWS accound ID

```
vault login -method=aws header_value=vault.example.com role=awsauth-role-iam aws_access_key_id=AKIATPLNME26BEGUOLUL aws_secret_access_key=N8F8L3CmthvxwHrGPjoALBjRe9khWFxrfbCma/y9
```
 ![Alt Image Text](../images/vt4_3_34.png "Body image")
 
```
$ vault login -method=aws header_value=vault.example.com role=awsauth-role-iam aws_access_key_id=<IAM_user_access_key> aws_secret_access_key=<IAM_user_secret_key>
```

## Azure and GCP Authentication Methods and Summary

* First enabled the Azure method, then we write its configuration under `auth/azure/config`.
*  We provide values for `tenant id`, resource, and most importantly, the `client_id` and `client_secret` that Vault will use to log in and make API calls against Azure. 
*  Finally, just as we did with AWS, we create the role involved where we specify the policies that will apply, the bound Azure subscription IDs and resource groups. 

> Enable Azure authentication in Vault:

```
$ vault auth enable azure
```

> Configure the Azure auth method:

```
$ vault write auth/azure/config \
	tenant_id=7cdlf227-ca67-4fc6-ala4-9888ea7f388c \
	resource=https://vault.hashicorp.com\
	client_id=dd794de4-4c6c-40b3-a930-d84cd32e9699 \
	client_secret=IT3B2XfZvWnfB98slcie8EMe7zWg483Xy8zY004=
```

For the complete list of configuration options, please see the API documentation.

> Create a role:

```
$ vault write auth/azure/role/dev-role
	policies="prod, dev"
	bound_subscription_ids=6a1d5988-5917-4221-b224-904cd7e24a25 \
	bound_resource_groups=vault
```

### Azure and AWS accounts and subscriptions

**Azure subscriptions are a grouping of resources with an assigned owner responsible for billing and permissions management.**

**Unlike AWS, where any resources created under the AWS account are tied to that account, subscriptions exist independently of their owner accounts, and can be reassigned to new owners as needed.**

 ![Alt Image Text](../images/vt4_3_35.png "Body image")

### Google 


IAM login applies only to roles of type iam . The Vault authentication workflow for AM service accounts looks like this:

 ![Alt Image Text](../images/vt4_3_36.png "Body image")

* 3 Create a named role:

```
$ vault write auth/gcp/role/my-iam-role \
	type="iam"
	policies="dev, prod"
	bound_service accounts="my-service@my-project.iam.gserviceaccount.com"
```

 
For a gce `-type` role:

```
$ vault write auth/qcp/role/my-gce-role\
	type="gce"\
	policies="dev, prod"
	bound_projects="my-projectl, my-project2" \
	bound_zones="us-eastl-b"\
	bound_labels="foo: bar, zip: zap"
	bound_service accounts="my-service@my-project.iam.gserviceaccount.com"
```

> Note that `bound_service_accounts` is only required for `iam` `-type` roles.

```
vault login -method=gcp \
	role="my-role"\
	service_account="authenticating-account@my-project.iam.gserviceaccount.com"\
	jwt_exp="15m" \
	credentials=apath/to/signer/credentials.json
```

## Summary

* Tokens and token types 
* Response wrapping 
* Traditional authentication methods 
* Cloud authentication methods 
* AppRole with Jenkins