# **L5 Using Vault Leases**

### Lease Overview

* Control dynamic secret lifecycle

> The purpose behind leases involved is to **control the lifecycle of dynamic secrets by either renewing or extending the lifetime of a dynamic secret** or by **revoking that dynamic secret by revoking the lease**.

* **Dynamic secrets and service tokens**

> Leases are a construct that actually exists for both **dynamic secrets and also for service tokens.** but for **secrets, they're called leases**.

* **Includes metadata about secret**

> The lease includes **some amount of metadata regarding** the secret and we'll get into what that information is in a moment.

* **Renew or revoke lease**

> You have the capability of extendina the lifetime of a lease bv renewing it or **you can terminate the lifetime of that lease by revoking it**.

* **No direct CLI command**
	* Use `/sys/leases/lookup` path

> There's **no direct CLI command like vault lease lookup**. 

### Examining Lease Duration

**Lease Properties**

* **`lease_id`**

The first is the lease_id, and this is a construct of the path where that **dynamic secret was generated**, and then a unique ID for that dynamic secret. 

* **`lease_duration`**

**The second main property is the `lease_duration`**. How much longer is this lease valid for?  **And that's basically a countdown timer**. It might start at 30 minutes and then start counting down. **If that lease duration expires**, then the lease is revoked, as well as the secret.

* **`lease_renewable`**

> The third property is whether or not the lease is renewable, and this is a Boolean value. 
> 
> Can you renew or extend this lease for longer, or is it a completely fixed time from when it's created?

### Working with Leases - **`lease_duration`**

* Time to live

> We can think of it in the same way that tokens have a TTL as well. and there's multiple sources where that default TTL can be derived.
> 
> 
>  The other important aspect about the lease is its max TTL,

* Max TTL

* TTL Inheritance
	* System
	* Mount
	* Object

> The first one is the system-wide default TTL and max TTL, and that's defined within the Vault server configuration. 
> 
> If default TTL and max TTL are not defined anywhere else, then the lease will inherit the values set for the system as a whole.
> 
> You can also configure the default and **max TTL on the mount point for a secrets engine**. precedence over what's set on the system,


* **Renewal**

### Working with Leases

* Renewal
	* Based on current time
	* Less than max TTL

* Revocation
	* Queues request
	* Token revokes leases

* Prefix Revocation
	* Requires sudo
	* Be careful

```
# Renew a lease

vault lease renew [options] ID 
vault lease renew -increment=30m consul/creds/web/KWq508RVc6LtAutsta6Uf86

# Revoke a lease
vault lease revoke [options] ID
vault lease revoke consul/creds/web/KWq508zRVc6LtAutsta6Uf86
vault lease revoke -prefix consul/creds/web/


# Lookup active leases
vault list [options] sys/leases/lookup/PATH
vault list sys/leases/lookup/consul/creds/web/

# View leases properties
vault write [options] sys/leases/lookup/ lease_id=ID
vault write sys/leases/lookup/lease_id=consul/creds/web/KWa508RVc6LtAutsta6Uf8G
```

### Lease Management Scenarios

**Use Case**

* Application needs credentials to access AWS resources
* Credentials should be revoked if application is inactive for 12 hours

**Solution**

* Enable an instance of AWS secrets engine
* Create AWS credentials for application with 12 hour lease duration
* Configure application to renew credentials

**Use Case**

* Multiple users need Consul tokens on a regular basis
* Tokens should expire after 60 minutes
* All tokens should be revoked if there is a credential breach

**Solution**

* Enable an instance of the Consul engine
* Create roles with a max lease TTL of 60 minutes
* Revoke leases with a prefix action

### Renewing a Lease with the CLI

**Pre Set**

```
# First of all we are going to start Vault in development mode
vault server -dev

# Now set your Vault address environment variable
export VAULT_ADDR=http://127.0.0.1:8200

# Set the root token variable
root_token=ROOT_TOKEN_VALUE

# And log into Vault using the root token
vault login $root_token

# Re-enable consul secrets engine
vault secrets enable consul

# Get consul up and running

# Create a data subdirectory in m8
mkdir data

# Launch consul server instance
consul agent -bootstrap -config-file="consul-config.hcl" -bind="127.0.0.1"

# From a separate terminal window run the following
consul acl bootstrap

# Set CONSUL_HTTP_TOKEN to SecretID

# Linux and MacOS
export CONSUL_HTTP_TOKEN=SECRETID_VALUE

# Next we have to create a policy and role for new tokens
# that Vault will generate on Consul

consul acl policy create -name=web -rules @web-policy.hcl

# Now we'll configure out Consul secrets engine
vault write consul/config/access address="http://127.0.0.1:8500" token=$CONSUL_HTTP_TOKEN
```

**generate a bunch of leases**

```
# And finally generate a bunch of leases to work with

vault read consul/creds/web
```

```
$ vault read consul/creds/web
Key                 Value
---                 -----
lease_id            consul/creds/web/vtO2IDHDLBfHK644D0V6sE81
lease_duration      1h
lease_renewable     true
accessor            07480394-784f-fcf0-d059-d96789b0136a
consul_namespace    n/a
local               false
partition           n/a
token               68796b0f-0ca8-5c6f-c04d-e410590fb00a

$ vault read consul/creds/web
Key                 Value
---                 -----
lease_id            consul/creds/web/hsQltEt7GFprYEb9pSvTTTBH
lease_duration      1h
lease_renewable     true
accessor            fb0cc137-f8de-5e0f-5022-dbbb69938fd9
consul_namespace    n/a
local               false
partition           n/a
token               bcf566f3-05d6-f73c-a3e2-5a72889b775e

$ vault read consul/creds/web
Key                 Value
---                 -----
lease_id            consul/creds/web/KeSvwCkIvY7iP8k7SqP6MQcD
lease_duration      1h
lease_renewable     true
accessor            e792364d-e232-13d2-01c7-4fc4c12b7caf
consul_namespace    n/a
local               false
partition           n/a
token               b79be0a8-fc68-6db9-6624-8d6371968a14

$ vault list sys/leases/lookup/consul/creds/web/
Keys
----
KeSvwCkIvY7iP8k7SqP6MQcD
hsQltEt7GFprYEb9pSvTTTBH
o4AT0MtgswjaSBNzYqgNGpxe
vtO2IDHDLBfHK644D0V6sE81
```

```
# Let's renew one of the leases for 30 minutes

vault lease renew -increment=30m LEASE_ID

$ vault lease renew -increment=30m consul/creds/web/KeSvwCkIvY7iP8k7SqP6MQcD
Key                Value
---                -----
lease_id           consul/creds/web/KeSvwCkIvY7iP8k7SqP6MQcD
lease_duration     30m
lease_renewable    true

# Now get the properties of the lease

vault write sys/leases/lookup lease_id=LEASE_ID

 vault write sys/leases/lookup lease_id=consul/creds/web/KeSvwCkIvY7iP8k7SqP6MQcD
Key             Value
---             -----
expire_time     2022-12-03T00:33:56.5601+08:00
id              consul/creds/web/KeSvwCkIvY7iP8k7SqP6MQcD
issue_time      2022-12-02T23:57:34.465805+08:00
last_renewal    2022-12-03T00:03:56.5601+08:00
renewable       true
ttl             29m20s
```

* **`ttl             29m20s`**

```
# What if we exceed the lease max ttl?
vault lease renew -increment=120m LEASE_ID

vault lease renew -increment=120m consul/creds/web/KeSvwCkIvY7iP8k7SqP6MQcD

$ vault lease renew -increment=120m consul/creds/web/KeSvwCkIvY7iP8k7SqP6MQcD
WARNING! The following warnings were returned from Vault:

  * TTL of "2h" exceeded the effective max_ttl of "1h51m5s"; TTL value is capped accordingly

Key                Value
---                -----
lease_id           consul/creds/web/KeSvwCkIvY7iP8k7SqP6MQcD
lease_duration     1h51m5s
lease_renewable    true
```

**`TTL of "2h" exceeded the effective max_ttl of "1h51m5s"; TTL value is capped accordingly`**


### Revoking Leases with the CLI

```
# Now we can try and revoke one of the leases
# First we'll get a list of active leases

vault list sys/leases/lookup/consul/creds/web/

# Now revoke the lease

vault lease revoke LEASE_ID

vault lease revoke consul/creds/web/KeSvwCkIvY7iP8k7SqP6MQcD

$ vault lease revoke consul/creds/web/KeSvwCkIvY7iP8k7SqP6MQcD
All revocation operations queued successfully!

$ vault list sys/leases/lookup/consul/creds/web/
Keys
----
hsQltEt7GFprYEb9pSvTTTBH
o4AT0MtgswjaSBNzYqgNGpxe
vtO2IDHDLBfHK644D0V6sE81

$ vault write sys/leases/lookup lease_id=consul/creds/web/KeSvwCkIvY7iP8k7SqP6MQcD
Error writing data to sys/leases/lookup: Error making API request.

URL: PUT http://127.0.0.1:8200/v1/sys/leases/lookup
Code: 400. Errors:

* invalid lease


# Confirm our lease is gone

# What if we want to revoke all of them? Prefix time

$ vault lease revoke -prefix consul/creds/web/
All revocation operations queued successfully!

# Confirm that all the leases are gone
 vault list sys/leases/lookup/consul/creds/web/
Keys
----
```

### Key Takeaways

* All dynamic secrets and service tokens have a lease that determines their validity period.
* **Lease duration can be extended by renewing the lease.** Renewals cannot exceed the maximum TTL.
* **Leases can be revoked before they expire using the Lease ID**. Revoking a token revokes all of its associated leases.
* **Multiple leases can be revoked usina a prefix**. which requires sudo permissions.
