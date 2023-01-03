# **L3 Integrating Vault with Terraform & Nomad**

## **1 Terraform Integration**

**Module Overview**

* Dynamic secrets in Terraform
* Nomad
  * TLS certificates
  * Access tokens

### Terraform

* **Codify infrastructure into declarative files**
* **Blueprint of infrastructure in source repos**
* Advantages
  * Resilience
  * Confidence
  * Security

### How Terraform Works

* Desired state
* Actual state
* API calls to match actual to desired

### A Terraform Workflow

![Alt Image Text](../images/chap5_3_1.png "Body image")

### Integration with Vault

* **Dedicated Terraform Vault provider**
* **Configuring Vault & populating secrets**

  * Can be challenging
  * Plaintext secrets in TF code
* **Using secrets from Vault**

  * Injecting auth data (trusted orchestrator)
  * API credentials for “plan” or “apply”
  * Provided token -> child token -> API creds
* **Separate read & write with Vault roles**

[https://bit.ly/vault-provider-docs](https://bit.ly/vault-provider-docs)

### Demo: Creating the Vault-Root AWS User

Let's create a policy that we can attach to the Vault root user.

[https://bit.ly/vault-aws-iam-policy](https://bit.ly/vault-aws-iam-policy)

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:AttachUserPolicy",
        "iam:CreateAccessKey",
        "iam:CreateUser",
        "iam:DeleteAccessKey",
        "iam:DeleteUser",
        "iam:DeleteUserPolicy",
        "iam:DetachUserPolicy",
        "iam:GetUser",
        "iam:ListAccessKeys",
        "iam:ListAttachedUserPolicies",
        "iam:ListGroupsForUser",
        "iam:ListUserPolicies",
        "iam:PutUserPolicy",
        "iam:AddUserToGroup",
        "iam:RemoveUserFromGroup"
      ],
      "Resource": ["arn:aws:iam::ACCOUNT-ID-WITHOUT-HYPHENS:user/vault-*"]
    }
  ]
}
```

* `"iam: GetUser"`
* `"Resource": ["arn: aws: iam: :089854689035 :user/vault-*"]`

![Alt Image Text](../images/chap5_3_2.png "Body image")

![Alt Image Text](../images/chap5_3_3.png "Body image")

```
$ vault secrets enable aws
Success! Enabled the aws secrets engine at: aws/

vault write aws/config/root\
> access_key=AKIARJ26BX4FSFCKYF60
> secret_key=EuQR206F5H0FHsGYpp98WmqPRu0fOzwakL5X1F2z\
> region=ca-central-1
Success! Data written to: aws/config/root
```

To rotate the credentials, use

**`vault write ‑force aws/config/rotate‑root`**

and Vault will echo back the new access key ID, but not the associated secret.

```
terraform vault write -force aws/config/rotate-root
Key   				Value
---   				----
access_key		AKIARJ26BX4FQZJFWAJ7
```

**`terraform-plan`**

```
vault write aws/roles/terraform-plan \
credential_type=iam_user\
policy_arns=arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess\
policy_document=@iam-getuser-self.json

Success! Data written to: aws/roles/terraform-plan
```

**`terraform-apply`**

```
vault write aws/roles/terraform-apply \
credential_type=iam_user \
policy_arns=arn:aws:iam::aws:policy/AmazonS3FullAccess \
policy_document-@iam-getuser-self.json


Success! Data written to: aws/roles/terraform-apply
```

```
vault write aws/config/lease lease=20m lease max=20m
Success! Data written to: aws/config/lease
```

```
vault policy write terraform-plan./terraform-plan.hcl

Success! Uploaded policy: terraform-plan
```

**`terraform-plan.hcl`**

```
path "aws/creds/terraform-plan" {
    capabilities = ["read"]
}
```

**`terraform-apply.hcl`**

```
path "aws/creds/terraform-apply" {
    capabilities = ["read"]
}
```

```
vault policy write terraform-apply./terraform-apply.hcl
Success! Uploaded policy: terraform-apply
```

**`token-creation.hcl`**

```
path "auth/token/create" {
    capabilities = ["update"]
}
```

```
vault policy write token-creation./token-creation.hcl
Success! Uploaded policy: token-creation
```

```
vault auth enable userpass
Success! Enabled userpass auth method at: userpass/
```

```
vault write auth/userpass/users/plan-user\
password=password \
policies=terraform-plan,token-creation

Success! Data written to: auth/userpass/users/plan-user


vault write auth/userpass/users/apply-user
password=password \
policies=terraform-apply,token-creation

Success! Data written to: auth/userpass/users/apply-user
```

```
VAULT_TOKEN=$(vault login \
-field=token -method=userpass \
username=plan-user password=password) \
vault read aws/creds/terraform-plan


Key     		value
---     		---
lease_id		aws/creds/terraform-plan/7jXup8HYnrBasPa@rbire@AI
lease_duration  20m
lease_renewable true
access_key      AKIARJ26BX4F7I7MSCUV 
secret_key		  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
security_token	<nils>
```

```
VAULT_TOKEN=$(vault login \
-field=token -method=userpass \
username=plan-user password=password) \
vault read aws/creds/terraform-apply

Error reading aws/creds/terraform-apply: Error making API request.

URL: GET http://127.0.0.1:8200/v1/aws/creds/terraform-apply
Code: 403. Errors:

* 1 error occurred:
	* permission denied
```

* `username=plan-user`
* `aws/creds/terraform-apply`

```
VAULT_TOKEN=$(vault login -field=token -method=userpass username=apply-user password=password) \
TF_VAR_AWS_CREDS_ROLE=terraform-apply 、
terraform apply
random_id.s3_postfix: Refreshing state... [id=0S7p2l21ojOLT0yg3iZ2fA]


VAULT_TOKEN=$(vault login -field=token -method=userpass username=apply-user password=password) \
terraform destroy
```

* Run `plan user` error
* `TF_VAR_AWS_REDS_ROLE=terraform-plan`

```
VAULT_TOKEN=$(vault login -field=token -method=userpass username=plan-user password=password) \
TF_VAR_AWS_REDS_ROLE=terraform-plan \
terraform apply
```

![Alt Image Text](../images/chap5_3_4.png "Body image")

* `username=plan-user`
* `TF_VAR_AWS_CREDS_ROLE=terraform-apply`

```

VAULT_TOKEN=$(vault login -field=token -method=userpass username=plan-user password=password) \
TF_VAR_AWS_CREDS_ROLE=terraform-apply \
terraform apply
```

![Alt Image Text](../images/chap5_3_5.png "Body image")

## Nomad Integration - Tokens

Nomad is HashiCorp's **workload orchestrator for deploying and managing software across cloud or data center environments at huge scale.**

HashiCorps core use cases for **Nomad are container orchestration, legacy application deployment, microservice management** when combined with HashiCorp console, batch processing of analytics workloads, **and building multi‑region or multi‑cloud deployments**

**Workload orchestrator**

* Deploying & managing software
* Cloud or data center at huge scale

**Core use cases**

* Container orchestration
* Legacy application deployment
* Microservice management (with Consul)
* Batch processing analytics workloads
* Multi-region or multi-cloud deployments

### **Kubernetes vs Nomad**

**Kubernetes**

* Built for containers
* Made from interoperating services
* Many community-built flavors
* Up to 5k nodes & 300k containers

**Nomad**

* Flexible workloads
* Single binary
* Consistent experience
* 10k nodes & 2M containers

### Nomad ACL System

* Built-in Access Control List (ACL) system
* Tokens to authenticate requests
* Policies define allowed actions
* Tokens are secrets & Vault can manage them
* Obtain token from Vault, use with Nomad

**`config.hcl`**

```
acl {
  enabled = true
}
```

```
nomad agent -dev -config=config.hcl

===> Loaded configuration from config.hcl
===> Starting Nomad agent...
```

### **Nomad Start**

```
$ nomad acl bootstrap

Accessor ID  = aedfa8a-acf0-8b3a-ddde-c1f81342f5da
Secret ID    = Odc11f1e-8cc4-3c62-3fb5-a885668a1bd
Name         = Bootstrap Token
Type         = management
Global       = true
Policies     = n/a
Create Time  = 2021-10-20 20:09:53.713725 +0000 UTC
Create Index = 11
Modify Index = 11
```

```
$ vault secrets enable nomad
Success! Enabled the nomad secrets engine at: nomad/

$ vault write nomad/config/access \
> address=http://localhost: 4646 \
> token=@dc11f1e-8cc4-3c62-3fb5-a88e5668a1bd

Success! Data written to: nomad/config/access

$ vault write nomad/role/management type=management global=true
Success! Data written to: nomad/role/management

$ vault read nomad/creds/management
Key     			Value
---     			---
lease_id   			nomad/creds/management/mnHLOLToCWOhLIXj6qn909rw 
lease_duration 		768h
lease_renewable 	true
accessor_id    		8810099-990-df3b-7a08-cb344ee97029
secret_id	   		a580d0af-7c11-ff6c-7e5b-303604cd03e7
```

```
nomad-tokensNOMAD_TOKEN=a580d0af-7c11-ff6c-7e5b-303604cd03e7\
> nomad acl token info 8810099-990-df3b-7a08-cb344ee97029
Accessor ID = 8810099-990-df3b-7a08-cb344ee97029
Secret ID   = a580d0af-7c11-ff6c-75b-303604cd03e7
Name        = vault-management-root-1634760866123581000
Type				= management
Global      = true
Policies    = n/a
Create Time = 2021-10-20 20:14:26.12459 +0000 UTC
Create Index = 17
Modify Index = 17
```

**`carved-rock-nomad-policy.hcl`**

```
namespace "carved-rock" {
  policy       = "write"
  capabilities = ["alloc-node-exec"]
}

agent {
  policy = "write"
}

operator {
  policy = "write"
}

quota {
  policy = "write"
}

node {
  policy = "write"
}

host_volume "*" {
  policy = "write"
}
```

```
NOMAD_TOKEN = 0dc11f1e-8cc4-3c62-3fb5-a88e5668a1bd\
> nomad ac policy apply carved-rock carved-rock-nomad-policy.hcl

Successfully wrote "carved-rock" ACL policy!
```

```
vault write nomad/role/carved-rock policies=carved-rock
Success! Data written to: nomad/role/carved-rock

vault read nomad/creds/carved-rock
Key		        Value
---       		----
lease_id		nomad/creds/carved-rock/Ureg8XTocZGS144AfZHcBDug
lease_duration  768h
lease_renewable  true
accessor_id		 75d0ca50-d8e7-bf51-46ab-4312542d463b
secret_id     87710770-4c15-16b6-2807-8a3b9ead3d87
```

```
NOMAD_TOKEN=87710770-4c15-16b6-2807-8a3b9ead3d87
> nomad acl token info 75d0ca50-d8e7-bf51-46ab-4312542d463b
Accessor ID = 75d0ca50-d87-bf51-46ab-4312542d463b
Secret ID   = 87710770-4c15-16b6-2807-8a3b9ead3d87
Name        = vault-carved-rock-root-1634761248999574000
Type        = client
Global      = false
Policies    = [carved-rock]
Create Time = 2021-10-20 20:20:49.000452 +0000 UTC
Create Index = 27
Modify Index = 27
```

[https://bit.ly/consul-template-github](https://bit.ly/consul-template-github)

## Nomad Integration - mTLS

### Nomad’s Communications

* HTTP between CLI & agents
* RPC between agents
* Serf between servers specifically
* “Agent” could be a client or server
* 3-5 servers, possibly 1000’s of clients
* Serf encrypted through a shared key
* HTTP & RPC mutual TLS (mTLS)
  * Vault can provide certificates

### Nomad mTLS

* Both sides verify the other’s identity
* Both sides provide certificates
* Different to TLS in (HTTPS), not mutual
* Prevent unauthorized access
* Prevent observing & tampering with comms
* Prevent misconfigurations
* Prevent masquerading

### mTLS Certificate Rotation

**Rotate mTLS certificates**

* On every node
* Use a short TTL
* Time consuming when done manually

**Consul-template**

* Retrieve secrets & update files
* Then run a command

### **Demo**

* PKI secrets engine
* Consul-template
* Automatically create & rotate certificates

## Demo: Configuring Vault & Consul-template

How to use **Vault's PKI secrets engine and Consul Template to automatically create and rotate certificates for mTLS in Nomad**.

**Setting up the root CA and intermediate CA**

It's mounted at `pki_int` on my Vault server.

* We'll start by creating a role in the intermediate CA for Nomad to generate certificates.
* The `max_ttl` is set to 24 hours.
* We need a Vault policy that will allow access to the issue endpoint for the role.

**`nomad-vault-policy.hcl`**

```
path "pki_int/issue/nomad" {
  capabilities = ["update"]
}
```

```
vault policy write nomad ./nomad-vault-policy.hcl

Success! Uploaded policy: nomad
```

> Note that the capability is update and not read.

### Consul Template

We need template files that will expand into an output and a configuration file to provide settings.

**`consul-template-config.hcl`**

The syntax of Consul Template allows you to query for a secret using this with secret string and then providing the path to the secret involved.

```
vault {
  address      = "http://localhost:8200"

  # You can use the Vault agent to manage the required Vault token
  # vault_agent_token_file = ""

  renew_token=false
}

# Server templates
template {
  source      = "server-templates/ca-certificate.template"
  destination = "server-secrets/ca.crt"
  # command     = ""
}

template {
  source      = "server-templates/certificate.template"
  destination = "server-secrets/server.crt"
  # command     = ""
}

template {
  source      = "server-templates/key.template"
  destination = "server-secrets/server.key"
  # command     = ""
}


# CLI templates
template {
  source      = "cli-templates/certificate.template"
  destination = "cli-secrets/cli.crt"
}

template {
  source      = "cli-templates/key.template"
  destination = "cli-secrets/cli.key"
}
```

Every Nomad agent, that is every Nomad server or client, will need a set of three **files to be generated, the issuing CA certificate, its own certificate**, and the associated private key.

```
├── client-templates
│   ├── ca-certificate.template
│   ├── certificate.template
│   └── key.template
```

**`ca-certificate.template`**

```
{{ with secret "pki_int/issue/nomad" "common_name=server.global.nomad" "ttl=24h"}}
{{ .Data.issuing_ca }}
{{ end }}
```

**`certificate.template`**

```
{{ with secret "pki_int/issue/nomad" "common_name=server.global.nomad" "ttl=24h" "alt_names=localhost" "ip_sans=127.0.0.1"}}
{{ .Data.certificate }}
{{ end }}
```

**`key.template`**

```
{{ with secret "pki_int/issue/nomad" "common_name=server.global.nomad" "ttl=24h" "alt_names=localhost" "ip_sans=127.0.0.1"}}
{{ .Data.private_key }}
{{ end }}
```

**On clients, the common name is `client.global.nomad`, and on servers, it is `server.global.nomad`. **

Notice as well that all of these templates are asking for their secrets with a TTL of 24 hours. Y

HashiCorp suggests adding these to every node in your cluster so that you can use the CLI anywhere within it.

```
cli-templates
│   ├── certificate.template
│   └── key.template
```

**`certificate.template`**

```
{{ with secret "pki_int/issue/nomad" "ttl=24h" }}
{{ .Data.certificate }}
{{ end }}
```

**`key.template`**

```
{{ with secret "pki_int/issue/nomad" "ttl=24h" }}
{{ .Data.private_key }}
{{ end }}
```

```
vault write pki int/roles/nomad allowed_domains=global.nomad \
allow_subdomains=true max_ttl=86400s require_cn=false generate_lease=true

Success! Data written to: pki_int/roles/nomad
```

```
consul-template-config=consul-template-config.hcl
```

![Alt Image Text](../images/chap5_3_6.png "Body image")

## Demo: Configuring Nomad for mTLS

**Configure nomad to use mTLS**

**`nomad-config.hcl`**

```
tls {
  http = true
  rpc  = true

  ca_file   = "server-secrets/ca.crt"
  cert_file = "server-secrets/server.crt"
  key_file  = "server-secrets/server.key"

  verify_server_hostname = true
  verify_https_client    = true
}
```

```
nomad agent -dev -config=nomad-config.hcl
=> Loaded configuration from nomad-config.hcl
=> Starting Nomad agent.
```

```
$ nomad status
Error querying jobs: Unexpected response code: 400 (Client sent an HTTP request to an HTTPS servr.
```

> we receive an error because the default nomad address for the CLI uses HTTP, and we've just configured the endpoint to use HTTPS

To make the CLI work, we need to export four environment variables.

```
export NOMAD_ADDR=https://localhost:4646
export NOMAD CACERT=server-secrets/ca.crt
export NOMAD CLIENT CERT=cli-secrets/cli.crt
export NOMAD CLIENT KEY=cli-secrets/cli.key
```

```
nomad status
No running jobs
```

```
openssl s_client -cert $NOMAD_CLIENT_CERT -key $NOMAD_CLIENT_KEY \
-connect localhost: 4646 2> /dev/null I
openssl x509 -noout -dates
notBefore=Oct 20 21:20:05 2021 GMT
notAfter=Oct 21 21:20:35 2021 GMT
```

**It connects to the Nomad endpoint using the CLI certificate and key, then grabs the certificate from the Nomad server and passes out the dates that the certificate is valid from and to.**

You can see here that the notBefore and not After date times make up a 24‑hour period

## Module Review

**Terraform**

* AWS secrets engine
* Differing levels of access for TF

**Nomad**

* Produce tokens
* PKI engine
* Consul-template
* mTLS
* Automatic certificate rotation
