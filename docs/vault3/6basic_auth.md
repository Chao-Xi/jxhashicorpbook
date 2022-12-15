# **L6 Configuring Basic Authentication**

## **Overview**

After you've initialized and unsealed your Vault cluster, you now have this root token that's kind of hanging around, and the root token's pretty dangerous

**So we'd like to replace that root token with some other form of authentication.**

* Review auth methods
* Review policies
* Globomantics requirements

## **Authentication Methods Review**

Vault Auth Methods

* **Provided by plug-ins**
* **Multiple methods allowed**


Authentication methods are provided by plugins, and there's **several plugins that are provided by default with the installation of Vault server**, but you can also load your own custom plugins for special authentication methods

* **Reference external sources**
	* - **LDAP, GitHub, AWS IAM, etc.**

Authentication methods refer to external sources of authentication, things you already have in your environment like LDAP or Active Directory, maybe you want to use GitHub accounts, or you could use AWS IAM accounts as your external source for authentication. 
	
	
* Token method is enabled by default

Vault will trust those sources to perform the authentication and then pass the authentication process back to Vault server. 

When you first install and configure Vault server, the only authentication method that's enabled is the token method. **And that's how the root token is generated**.

* **Used to obtain a token**


When a client wants to interact with your Vault cluster, they're going to need a token. 

Vault will **trust those sources to perform the authentication and then pass the authentication process back to Vault server**. 


When you first install and configure Vault server, the only authentication method that's enabled is the token method. **And that's how the root token is generated**. 

All the other authentication methods in Vault are really just there to obtain a token, and then the token is used for all subsequent requests to the Vault cluster. 

### Username & Password

* Meant for human operators
* Composed of a username and password
* Internal to Vault

* The username and password authentication method is available on Vault server, and it's meant for human operators, someone like a Vault administrator perhaps. 
* The nice thing about username and password, which is also called **userpass**, is that **it is internal to Vault**. 
* It doesn't rely on an external authentication source. Which means if something happens to your external authentication source, **say Active Directory is down or you can't get out to AWS IAM, you can still use userpass to authenticate to Vault.** 

## Authentication Method Configuration

* All methods are enabled on `/sys/auth`
* Methods are enabled on a path
	* Defaults to method name
* **Methods cannot be moved**

You could disable it and reenable it on a new path, but you'll lose all the data that was stored at the original path for the authentication method.

* **Methods can be tuned and configured**
	* Tuning settings are common for all methods
	* Configuration settings are specific to a 
method
	
An important thing to understand about authentication methods is that they cannot be moved after they've been enabled. 

**You can't change the path where your authentication method is**. 

Configuration settings are specific to a method. **For instance, the settings you need for the AWS IAM authentication method are going to be a little bit different than what you use for LDAP**, and those are configured with the write command along the path for the authentication method.


### **Auth Method Commands**

```
# List existing auth methods
vault auth list

# Enable an auth method
vault auth enable [options] TYPE
vault auth enable –path=globopass userpass


# Tune an auth method
vault auth tune [options] PATH
vault auth tune –description="First userpass" globopass/

# Disable an auth method
vault auth disable [options] PATH
vault auth disable globopass/
```


### **Dev Requirements**

* Revoke root tokens as soon as possible
* Well defined permissions for Vault admins

### **Auth Methods Demo Overview**


```
# Set your Vault address environment variable
# Ex. vault-vms.globomantics.xyz
export VAULT_ADDR=https://VAULT_SERVER_FQDN:8200

# And log into Vault using the root token
vault login 

Key     				 Value
---     				 --- 
token   				 s.lTiyARrCtkILBpMpJROhS418
token_accessor    N7SOwDyvdVLYpKqLLBbiuDN3
token_duration   
token_renewable  false
token_policies   ["root"]
identity_policies []
policies           ["root"]
```

```
# First let's see what auth methods are avilable now
vault auth list

$vault auth list
Path	   Type  	 Accessor            Description
token/  token  auth_token_b8c48497  token based credentials
```

```
# Cool, now let's enable our first auth method using userpass
vault auth enable userpass


# Now let's check the list of auth methods again
$vault auth list
Path	   Type  	 Accessor            Description
token/     token  auth_token_b8c48497  token based credentials
userpass/ userpass auth_userpass_b6134075 n/a

# add descriptions
vault auth tune -description="Globomantics Userpass" userpass/
Success! Tuned the auth method at: userpass/

# Now let's check the list of auth methods again
$vault auth list
Path	   Type  	 Accessor            Description
token/     token  auth_token_b8c48497  token based credentials
userpass/ userpass auth_userpass_b6134075 Globomantics Userpass
```

```
# Let's write some data to create a new user

vault write auth/userpass/users/testadmin password=testpassword
```

## **Vault Policies Review**

**<mean>Vault policies are the means by which permissions are assigned to tokens.</mean>**

* **Policies define permissions in Vault**
* **Multiple options for assignment**
	* - Token, identity, auth methods

Vault policies are what defines permissions in Vault, and you have multiple options for where policies can be assigned. 

You can **assign a policy at the token level, you can use the internal Identity secrets engine or you can assign it within one of the authentication methods** that you've enabled	
	
	
* Most specific wins

If you have a situation where a token has multiple overlapping policies assigned to it, the most specific policy wins. 

* No versioning

Another important thing to note about Vault policies is that there's no versioning of policies. When you update a policy in Vault, it doesn't write a new version of the policy, it overwrites the existing policy in place.

So if you're planning to version your policies, you're going to have to do that outside of the context of Vault


* Default policy

> The default policy can be edited, but it cannot be deleted. 

* Root policy

The other special policy is the root policy. **The root policy is what's assigned to the root token, and it can do anything. You also cannot edit the root policy, and you cannot delete it.**


### Policy Syntax

* **HCL or JSON**

Vault policies are written in either HCL or in JSON.

* **Path**

The first one is a path. The path defines where you're specifying permissions in Vault server.

* Capabilities

Within the configuration block for the path, you specify the capabilities that you want to allow or deny for that path. 


## Working with Policies

```
# List existing policies
vault policy list

# Read the contents of a policy
vault policy read [options] NAME
vault policy read secrets-mgmt

# Write a new policy or update an existing policy
vault policy write [options] NAME PATH | <stdin>
vault policy write secrets-mgmt secrets-mgmt.hcl

# Delete a policy
vault policy delete [options] NAME
vault policy delete secrets-mgmt

# Format a policy per HCL guidelines
vault policy fmt [options] PATH
vault policy fmt secrets-mgmt.hcl
```

### Polices Demo

* Create an admin policy
* Assign the admin policy
* Revoke the current root token


## Assigning the Admin Policy

```
# First we can check and see what policies exist right now
$ vault policy list 
default
root
```

Now we'll create a policy for Vault administration 

This policy is based off an example provided by HashiCorp

**`vault-admins.hcl`**

```
# Allow managing leases
path "sys/leases/*"
{
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# Manage auth backends broadly across Vault
path "auth/*"
{
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# List, create, update, and delete auth backends
path "sys/auth/*"
{
  capabilities = ["create", "read", "update", "delete", "sudo"]
}

# List existing policies
path "sys/policies"
{
  capabilities = ["read"]
}

# Create and manage ACL policies broadly across Vault
path "sys/policies/*"
{
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# List, create, update, and delete key/value secrets
path "secret/*"
{
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# Manage and manage secret backends broadly across Vault.
path "sys/mounts/*"
{
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# List existing secret engines.
path "sys/mounts"
{
  capabilities = ["read"]
}

# Read health checks
path "sys/health"
{
  capabilities = ["read", "sudo"]
}
```

```
vault policy write vault-admins vault-admins.hcl

$ vault policy list
default
vault-admins
root

# Next we can assign the policy to the globoadmin user
vault write auth/userpass/users/testadmin token_policies="vault-admins"

# Now we can log in as testadmin and a few actions
vault login -method=userpass username=testadmin

# List all secrets engines
$ vault secrets list

Path       Type      Accessor     Description
cubbyhole/ cubbyhole cubbyhole_06c1ad2e per-token private secret storage
identity/  identity  identity_5b5760de  identity store
sys/       system    system_a66b4ae1    system endpoints used for control, policy and debugging


# Enable a secrets engine
vault secrets enable -path=testing -version=1 kv

$ vault secrets list
Path       Type      Accessor     Description
cubbyhole/ cubbyhole cubbyhole_06c1ad2e per-token private secret storage
identity/  identity  identity_5b5760de  identity store
sys/       system    system_a66b4ae1    system endpoints used for control, policy and debugging
testing/   kv       kv_f80de1d4    n/a



# Let's revoke the current root token 
# and use the admin account from here on out
vault token revoke ROOT_TOKEN

$ vault token revoke s.ITiyARrCtkILBpMpJROhS418
Success! Revoked token (if it existed)

$vault login
Token (will be hidden):
Error authenticating: error looking up token: Error making API request.
```

## **Module Summary**

* Auth methods are used to obtain a Vault token
* Policies define the permissions associated with a token
* Root tokens are not meant for administrative work