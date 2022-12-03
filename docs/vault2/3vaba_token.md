# **L3 Vault Token**

* Token overview
* Properties and attributes
* Token types
* Token lifecycle

## Vault Token Overview

Tokens are a collection of data used to access Vault


### **Token Creation**

* **Auth method**

* **Parent token**
	* You can also use an existing token to generate a new token. You can use the root token that's created when you initialize Vault to create a child token.
	* You can also use a token that's been issued by an auth method

* **Root token**: you can generate a root token through a special process.

### **Root Tokens**

* Root tokens can do ANYTHING
* Do not expire
* Created in three ways
	* **Initialize Vault server**
	* **Existing root token**
		* You can also use an existina root token to create a new root token.	
	* **Using operator command**


What are the reasons you would generate a root token?

* Perform initial setup
* Auth method unavailable
* Emergency situation


You may also generate a root token if your main authentication method is unavailable. Let's say you use Active Directory for your main authentication method, and Active Directory, for whatever reason, **You might have to generate a root token to enable a different authentication method**. 	


### **Token Properties**

* **Id**

This is the value that you will submit when you want to perform an action with Vault whether it's logging in as that token or making use of the Vault API.

* **Accessor**

There's another related idea that's called the accessor,  behind the accessor is that it's a value you can use to look up the token without actually being able to use that token.

* **Type**

Defines what type of token it is, and generally it's either **service** or **batch**,

* **Policies**

The token will also have policies associated with it. Those policies can be assigned when the token is issued or they could be continually evaluated when the token is being used.

* **TTL**

Another important property is the Time to Live, or TTL. This defines how long is this token going to be a viable token

* **Orphaned**

Determines whether or not this token has a parent token or if it is a standalone token.


### Accessor 

**ID and Accessor**

* **View token properties except token ID**
	* The accessor is only able to view the token properties
	* The ID is the value you would use to execute actions against Vault server	* accessor to also be able to get that ID.

* **View token capabilities on a given path**
	* **capabilities** of a token on a given path. So if you want to check if a token is able to access a path within Vault

* **Renew or revoke a token**
	* accessor to renew an existing token or revoke a token


What are some situations where you might just want that accessor value?
	
* **Parent process controlling child tokens**
	* You don't want that parent process to have the ID of those child tokens. Those are given to whatever workers are below that process.
* **View accessors at auth/token/accessors**
	* you just want that process to have the accessor so it has the capability to revoke and check on the status of existing child tokens.
	* You may also want to get a list of all tokens that have been issued. **That's something you can do by viewing the accessors that are at auth/token/accessors.**
	
* **Audit token usage by accessor in audit log** 
	* you can configure the audit log to log the accessor value for each action taken by a token. **You obviously wouldn't want to log the token ID, but the accessor gives**


### Working with tokens

```
# Create a new token
vault token create [options]
vault token create -policy=my-policy -ttl=60m

# View token properties
vault token lookup [options] [ ACCESSOR | ID ]
vault token lookup -accessor FJkyU351hsMf3nKOLWdouqdY

# Check capabilities on a path
vault token capabilities TOKEN PATH
vault token capabilities s.TG9U2ZdtPU1Hmz18BcujrETI secrets/apikeys/
```

### Creating with tokens with CLI

* Create Vault service token
* Obtain tokens from auth methods
* Create a batch token
* Renew and revoke tokens
* Create a periodic token

```
root_token=ROOT_TOKEN_VALUE

root_token=hvs.VtsTEZn9wbP34P63WybIZMhv

$ echo $root_token
hvs.VtsTEZn9wbP34P63WybIZMhv

# And log into Vault using the root token
vault login $root_token


$ vault login $root_token
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                hvs.VtsTEZn9wbP34P63WybIZMhv
token_accessor       aONrVciIIfKgvuvbeSrcXV4u
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]


# First we are going to create a basic token

vault token create -policy=default -ttl=60m


$ vault token create -policy=default -ttl=60m
Key                  Value
---                  -----
token                hvs.CAESIJ3n0vyQcj7nYijaTUbvmliqnmGFq3feFsY7_wGJzpu3Gh4KHGh2cy5NMkYzNmhCYmcxZ2FTT0cwOTBSVHpJRHg
token_accessor       wsp4jbhoFtTYw2FwbUliWCMB
token_duration       1h
token_renewable      true
token_policies       ["default"]
identity_policies    []
policies             ["default"]


# Now let's check out some info on the token

vault token lookup hvs.CAESIJ3n0vyQcj7nYijaTUbvmliqnmGFq3feFsY7_wGJzpu3Gh4KHGh2cy5NMkYzNmhCYmcxZ2FTT0cwOTBSVHpJRHg


$ vault token lookup hvs.CAESIJ3n0vyQcj7nYijaTUbvmliqnmGFq3feFsY7_wGJzpu3Gh4KHGh2cy5NMkYzNmhCYmcxZ2FTT0cwOTBSVHpJRHg
Key                 Value
---                 -----
accessor            wsp4jbhoFtTYw2FwbUliWCMB
creation_time       1668088568
creation_ttl        1h
display_name        token
entity_id           n/a
expire_time         2022-11-10T22:56:08.071587+08:00
explicit_max_ttl    0s
id                  hvs.CAESIJ3n0vyQcj7nYijaTUbvmliqnmGFq3feFsY7_wGJzpu3Gh4KHGh2cy5NMkYzNmhCYmcxZ2FTT0cwOTBSVHpJRHg
issue_time          2022-11-10T21:56:08.071594+08:00
meta                <nil>
num_uses            0
orphan              false
path                auth/token/create
policies            [default]
renewable           true
ttl                 55m6s
type                service
```

* ttl： 55m6s

```
# We can do the same using the accessor, but no ID

vault token lookup -accessor wsp4jbhoFtTYw2FwbUliWCMB

 vault token lookup -accessor wsp4jbhoFtTYw2FwbUliWCMB
Key                 Value
---                 -----
accessor            wsp4jbhoFtTYw2FwbUliWCMB
creation_time       1668088568
creation_ttl        1h
display_name        token
entity_id           n/a
expire_time         2022-11-10T22:56:08.071587+08:00
explicit_max_ttl    0s
id                  n/a
issue_time          2022-11-10T21:56:08.071594+08:00
meta                <nil>
num_uses            0
orphan              false
path                auth/token/create
policies            [default]
renewable           true
ttl                 51m51s
type                service

# Now let's revoke our token
$ vault token revoke -accessor wsp4jbhoFtTYw2FwbUliWCMB
Success! Revoked token (if it existed)
```

### **Token Types and Lifecycle**


There are different token types out there, **like root, service, batch, and even periodic**.


**Service or Batch**

**Service**

* **Fully featured, heavyweight**
	* A service token is fully featured, but the downside is that it's a little bit heavyweight.
	* All service tokens have to be written to the storage back end.
* **Managed by accessor or ID**
	* The service tokens can be managed by the accessor or the ID.
* **Written to persistent storage**
	* The service token is written to persistent storage,
* **Calculated lifetime**
	* A service token has a calculated lifetime that's based off of the TTL
	* Any renewals that you do against that token,
* **Renewable if desired**
	* Service tokens can be renewed if desired.
	* You can set them to non-renewable, but you can enable renewal if you want,
* **Can create child tokens**
	* 	Service tokens can spawn child tokens of the type either service or batch.
* **Default type for most situations**
* Begins with "s." in ID


**Batch**

* **Limited features, lightweight**
	* Batch tokens have a limited feature set. but they're also very lightweight in the sense that
	* They're not written to the storage back end.
	* They can be generated and managed very quickly.
* **Has no accessor**
	* The service tokens can be managed by the accessor or the ID.
	* The batch token, **because it's not being written to persistent storage, doesn't have an accessor.**
	* **If you don't have the batch token ID, you don't have a way to manage it**.
* **Not written to storage**
	* **The batch token is held in memory until it expires**.
* **Static lifetime**
	* batch token has a static lifetime.
	* **The TTL of the batch token is set during creation and cannot be altered**. 
* **Never renewable**
	* batch tokens are not renewable.
	* Once they hit their Time to Live, they are gone.
* **No child tokens**
	* Batch tokens are not able to create any child tokens.
	* They can't spawn additional tokens from themselves.
* **Explicitly created**
	* You have to be explicit about that. When you're looking at the ID for	
* **Begins with "b." in ID**


### **Batch token scenario**

```
# Token TTL
creation_time  1613828388 # Unix time
creation_ttl   30m # TTL set at creation

```

This is the TTL that was set when we ran the vault token create or **when it was generated from an authentication method**.

```
expire_time    2021-02-20T09:09:48.4036711-05:00  # Project expiration time

explicit_max_ttl  2021-02-20T08:39:48.4036711-05:00 # Friendly creation time

ttl    29m13s # TTL value
```

This is what vault is projecting as the expiration time if nothing is changed about this token. 

There is an `explicit_max_ttl` field, which can be set when you create the token that **will set a maximum time to live for friendly representation of the creation time in non-UNIX** time 

we have the current TTL, and this is a computed value that is given to

### **Working with Token Lifetime**

```
# Renew a token
vault token renew [options] [ACCESSOR | ID ] [ -increment=<duration> ]
vault token renew -increment=60m

# Revoke a token

vault token revoke [options] |[ACCESSOR | ID ]
vault token revoke -accessor FJkyU351hsMf3nKOLWdOUqdY
```

### Assessing the Effective Max TTL

**System max TTL**

* System wide setting
* Vault configuration file
* Dynamic evaluation


> If you don't set it in the Vault configuration file, then Vault will choose a value for you, **which is 32 days.**
> 
> The system max TTL is subject to dynamic evaluation, meaning each time Vault assesses the lifetime of a token, it's going to check what the current system max TTL is. 

And if it changes, more specific place where you can set the value for the max TTL is on the mount of an authentication method.

**Mount max TTL**

* Mount specific
* Change with tuning
* Override system max


You could have multiple authentication methods, The way that you change that value is with the **Vault tune command**.

When you set a value for the mount max TTL, it will override whatever the setting is in the system max TTL, and **that value can be greater than or less than the system max TTL**

If your system max TTL is 32 days and you set the mount max TTL to 34 days, **that will be the effective max TTL for that authentication method.**

**Auth method max TTL**

* Role, group, user
* Changed with write
* Override system or mount max
* Less than system or mount

be less than either the system max TTL or the mount max TTL


### Exploring the Effective Max TTL

```
$ vault server -dev
Root Token: hvs.XV2DORivLvdQnW3PIKdi1yD1


export VAULT_ADDR='http://127.0.0.1:8200'
root_token=hvs.XV2DORivLvdQnW3PIKdi1yD1
vault login $root_token

...
$ vault login
Token (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                hvs.XV2DORivLvdQnW3PIKdi1yD1
token_accessor       2vW6uauAn2XYIGlGlStN3N5V
```

try setting max ttl from the mount and user and Start by enabling the max ttl for userpass to 33 days (776h)

```
# Enable userpass auth method
vault auth enable -max-lease-ttl=776h userpass

...
Success! Enabled userpass auth method at: userpass/
```

Now we are going to try and configure a user with a great max ttl of 784h

Note: Vault will let you do this, but it won't honor it.

```
vault write auth/userpass/users/dev token_max_ttl=2822400 password=ops

Success! Data written to: auth/userpass/users/dev


# Let's try logging in as Dev and renewing our token for 34 days (784h)

vault login -method=userpass username=dev

 vault login -method=userpass username=dev
Password (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                    Value
---                    -----
token                  hvs.CAESIND6s_AzTvwx_R4lkxlEh6bN1yWpRn7SG8xKlNe12KoQGh4KHGh2cy54QVhtME9SYjhSblVyZFgxaGY2OEZyQWk
token_accessor         g14HUvVx8yLz96jaNyZ5pj9Z
token_duration         768h
token_renewable        true
token_policies         ["default"]
identity_policies      []
policies               ["default"]
token_meta_username    dev
```

```
vault token renew -increment=784h

# If we changed the max TTL on the mount to 784h, then Vault would honor 
# the value set at the user level. I leave this as an exercise for you.

$ vault token renew -increment=784h
WARNING! The following warnings were returned from Vault:

  * TTL of "784h" exceeded the effective max_ttl of "775h58m1s"; TTL value is
  capped accordingly

Key                    Value
---                    -----
token                  hvs.CAESIND6s_AzTvwx_R4lkxlEh6bN1yWpRn7SG8xKlNe12KoQGh4KHGh2cy54QVhtME9SYjhSblVyZFgxaGY2OEZyQWk
token_accessor         g14HUvVx8yLz96jaNyZ5pj9Z


# Now we can revoke our own token if we're done with our work

$ vault token revoke -self
Success! Revoked token (if it existed)

$ vault login $root_token

$ vault token lookup
Key                 Value
---                 -----
accessor            2vW6uauAn2XYIGlGlStN3N5V
creation_time       1668613087
creation_ttl        0s
display_name        root
entity_id           n/a
expire_time         <nil>
explicit_max_ttl    0s
id                  hvs.XV2DORivLvdQnW3PIKdi1yD1
meta                <nil>
num_uses            0
orphan              true
path                auth/token/root
policies            [root]
ttl                 0s
type                service
```

### **Explicit Max TTL**

* Takes precedence
* Set at token creation
	* Explicitly in command
	* Implicitly through configuration
* Static evaluation
* Less than effective max TTL


### **Introducing the periordic Token**

TTL defines the period in which that token needs to be renewed

* **Does not expire (no max TTL)**
* **Must be renewed based on period**
* **TTL set to period at creation and renewal**
* **Requires sudo privileges to create**
* Explicit max TTL can be applied


You could set a token with a period of 10 minutes, meaning that whatever service is using that token would have to renew the token within 10 minutes or that token will expire. 

Because the periodic


**Use Case**

* Database system will use token for secrets access
* System does not support dynamically changing the token value

**Solution**

* Create a periodic token for the database system to use
* **Script a process to renew the token at the necessarv interval**

```
# Login as the root token

vault login $root_token

# Now create a periodic token

vault token create -policy=default -period=2h

Key                  Value
---                  -----
token                hvs.CAESICXVwYvLCNiZUlgx1r2fge8Mh8JEMKYaMTTJ4jLE60baGh4KHGh2cy44NzJFUEppVWt0cndCMDhud0ZGVEVqeTk
token_accessor       kx9ddr7d178SkHN3w8nkbCyP
token_duration       2h
token_renewable      true
token_policies       ["default"]
identity_policies    []
policies             ["default"]


export PERIODIC_TOKEN_ID=hvs.CAESICXVwYvLCNiZUlgx1r2fge8Mh8JEMKYaMTTJ4jLE60baGh4KHGh2cy44NzJFUEppVWt0cndCMDhud0ZGVEVqeTk

# And take a look at its properties

vault token lookup $PERIODIC_TOKEN_ID


$ vault token lookup $PERIODIC_TOKEN_ID
Key                 Value
---                 -----
accessor            kx9ddr7d178SkHN3w8nkbCyP
creation_time       1668644449
creation_ttl        2h
display_name        token
entity_id           n/a
expire_time         2022-11-17T10:20:49.068383+08:00
explicit_max_ttl    0s
id                  hvs.CAESICXVwYvLCNiZUlgx1r2fge8Mh8JEMKYaMTTJ4jLE60baGh4KHGh2cy44NzJFUEppVWt0cndCMDhud0ZGVEVqeTk
issue_time          2022-11-17T08:20:49.06839+08:00
meta                <nil>
num_uses            0
orphan              false
path                auth/token/create
period              2h
policies            [default]
renewable           true
ttl                 1h55m14s
type                service


# Now let's try to renew

vault token renew -increment=180m $PERIODIC_TOKEN_ID


$ vault token renew -increment=180m $PERIODIC_TOKEN_ID
Key                  Value
---                  -----
token                hvs.CAESICXVwYvLCNiZUlgx1r2fge8Mh8JEMKYaMTTJ4jLE60baGh4KHGh2cy44NzJFUEppVWt0cndCMDhud0ZGVEVqeTk
token_accessor       kx9ddr7d178SkHN3w8nkbCyP
token_duration       2h
token_renewable      true
token_policies       ["default"]
identity_policies    []
policies             ["default"]


$ vault token lookup $PERIODIC_TOKEN_ID


$ vault token lookup $PERIODIC_TOKEN_ID
Key                  Value
---                  -----
accessor             kx9ddr7d178SkHN3w8nkbCyP
creation_time        1668644449
creation_ttl         2h
display_name         token
entity_id            n/a
expire_time          2022-11-17T10:27:13.822212+08:00
explicit_max_ttl     0s
id                   hvs.CAESICXVwYvLCNiZUlgx1r2fge8Mh8JEMKYaMTTJ4jLE60baGh4KHGh2cy44NzJFUEppVWt0cndCMDhud0ZGVEVqeTk
issue_time           2022-11-17T08:20:49.06839+08:00
last_renewal         2022-11-17T08:27:13.822212+08:00
last_renewal_time    1668644833
meta                 <nil>
num_uses             0
orphan               false
path                 auth/token/create
period               2h
policies             [default]
renewable            true
ttl                  1h59m44s
type                 service
```

**Looking at the properties again, the ttl is back to 2h If you supply an increment, Vault ignores it**


### Token Hierarchy

* Child tokens are created by a parent token
* Batch tokens cannot create children
* Protects against escaping revocation
* **Orphan tokens have no parent token**
	* Explicit creation
	* Auth methods
	* **Orphaned by parent**

when a parent token is revoked all of the children token associated with that parent token will also be revoked,

So if you create a token for someone and they're trying permissions and use that instead,


### **Key Takeaways**

* Token TTL is the amount of time a token is valid for. Tokens can be renewed for additional time within the effective max TTL.
* Periodic tokens can be renewed forever based on a period TTL Require elevated permissions and may have an explicit max TTL
* Tokens have a hierarchy of parent/child. Revoking a parent token revokes the children by default. Orphaned tokens have no parent.