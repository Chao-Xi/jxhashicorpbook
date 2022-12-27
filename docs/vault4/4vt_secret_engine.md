# **L4 Managing Secrets with Secrets Engines**

* Database secrets engine at Globomantics
* SSH secrets engine - manage SSH servers
* PKI secrets engine - run your own Certificate Authority

## **Using the Database Secrets Engine**

![Alt Image Text](../images/vt5_1_1.png "Body image")

### Vault Database Secrets Engines

* Don’t store secrets in relational databases 
* Function as simple authentication mechanisms

### **Distribute Username / Password Secrets**

![Alt Image Text](../images/vt5_1_2.png "Body image")

### Using Vault to Generate Dynamic Credentials

![Alt Image Text](../images/vt5_1_3.png "Body image")

### Best Practices

* Dynamic secrets 
* Leases to control secrets lifecycle

### **Demo**

Using the Database Secrets Engine

* Enable DB secrets engine 
* Configure its database connection 
* Set roles in the database secrets engine 
* Create related policies in Vault 
* Generate tokens 
* Use tokens to log into the database

![Alt Image Text](../images/vt5_1_4.png "Body image")

```
$ vault server -dev
...
You may need to set the following environment variables:

    $ export VAULT_ADDR='http://127.0.0.1:8200'

The unseal key and root token are displayed below in case you want to
seal/unseal the Vault or re-authenticate.

Unseal Key: H65WnFroE99hEp17qQ/qYG8+T1TF/6seryMoG6q52PI=
Root Token: hvs.mkzvtZeivMkg6SSpdA3Wfk1m
...



$ vault operator unseal H65WnFroE99hEp17qQ/qYG8+T1TF/6seryMoG6q52PI=
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.12.2
Build Date      2022-11-23T12:53:46Z
Storage Type    inmem
Cluster Name    vault-cluster-f56d0349
Cluster ID      4694a9ba-acf0-89b2-98f3-90657f4d52af
HA Enabled      false
```

### install MySQL

```
# Install MySQL
brew install mysql

# Start the database - runs as a daemon
brew services start mysql

# To start it on the foreground
mysql.server start
mysql.server stop


# Secure the database - by default there is not root password
mysql_secure_installation
# Press y|Y for Yes, any other key for No: Type N, as this database is just for testing purposes, and thus you don't need to test password strength
# Remove anonymous users? (Press y|Y for Yes, any other key for No) : again type N
# Disallow root login remotely? (Press y|Y for Yes, any other key for No) : type N
# Remove test database and access to it? (Press y|Y for Yes, any other key for No) : type N
# Reload privilege tables now? (Press y|Y for Yes, any other key for No) : type Y
# Success.
# All done!

# Test the installation
mysql -u root -p   
```

### Enable vailt database

```
# Enable the database secrets engine
vault secrets enable database

$ vault secrets enable database
Success! Enabled the database secrets engine at: database/


# Configure the db engine
vault write database/config/mysql \
    plugin_name=mysql-database-plugin \
    connection_url="{{username}}:{{password}}@tcp(localhost:3306)/" \
    allowed_roles="reader,writer" \
    username="root" \
    password="sup3rm1sqlp@ss0rd"
    

# Create reader role
vault write database/roles/reader \
    db_name=mysql \
    creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}'; GRANT SELECT ON *.* TO '{{name}}'@'%';" \
    default_ttl="2h" \
    max_ttl="48h"
    

# Create writer role
vault write database/roles/writer \
    db_name=mysql \
    creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}'; GRANT ALL ON *.* TO '{{name}}'@'%';" \
    default_ttl="2h" \
    max_ttl="48h" 
```

* **`reader.hcl`**

```
# Allow the reader role access to the path that generates the credentials
path "database/creds/reader" {
    policy = "read"
}
```

* **`writer.hcl`**

```
# Allow the writer role access to the path that generates the credentials
path "database/creds/writer" {
    policy = "read"
}
```


``` 
# Upload the policies
vault policy write reader reader.hcl
Success! Uploaded policy: reader


vault policy write writer writer.hcl
Success! Uploaded policy: writer
```

### Reader role with read policy

```
# Create tokens, log in, and create credentials with the reader role
$ vault token create -policy=reader

$ vault token create -policy=reader

Key       Value
token	 S.CMFwtkvmgubnhzLVLgBARAP9
token_accessor	deELtZVqUfXrEjmMYFBLWy6M
token_duration	768h
token_renewable	true
token_policies	[“default" “reader”]
identity_policies	[]
policies		["default”“reader”]
```


```
# Create tokens, log in, and create credentials with the reader role
vault token create -policy=reader

$ vault login s.CMFWtkvmgubnhzlVLgBARAP9

$ vault read database/creds/reader
Key   Value
Lease_id	database/creds/reader/va4tpxgvbgSiQqkoE476BX2
Lease_duration	2h
Lease_renewable	true
password	nJsaAAZtO-CLOpjIUwZV
username	V-token-reader-20mJBKAAIg4AKWKOy
```

![Alt Image Text](../images/vt5_1_5.png "Body image")

```
mysql -uv-token-reader-20mJBKAAIg4AKwKOy -pnJsaAAZto-CL0pjIUwzV -h localhost

mysql> show databases;
+-------—---------—--+
| Database           |
+-------—---------—--+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+-------—---------—--+
4 rows in set (0.00 sec)

mysql» use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

mysql» create database globomanticsdb;
ERROR 1044 (42000): Access denied for user 'v-token-reader-20mJBKAAIg4AKwKOy'a'%' to database 'globomanticsdb
```

### Writer role with write policy

```
# as root
$ vault login hvs.mkzvtZeivMkg6SSpdA3Wfk1m
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                hvs.mkzvtZeivMkg6SSpdA3Wfk1m
token_accessor       sL3wVBYzguqLCVVuaDPjl0ur
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]

$ vault token create -policy=writer

Key				Value
token			s.9i1yhhTocogoR7WULCVQ02
token_accessor	KVvQYY9tX6bd4QkYL4dvtp
token_duration	768h
token_renewable	true
token_poicies	[“default”"writer"]
identity_policies []
policies	[“default" "writer]
```

```
vault login s.9i1yhhTocogoR7WULCVQ02

vault read database/creds/writer
Key	Value
Lease_id	database/creds/writer/Ic10U9KzL5ECCNmN4gDILjjL
Lease_duration	2h
Lease_renewable	true
password	nRG7iHGRC6-dVhru5uvh
username	v-token-writer-x43SOMOPjCEL6Zvkb
```

![Alt Image Text](../images/vt5_1_6.png "Body image")


Success; the writer role and its corresponding user have create (ALL) privileges


```
# Now log in to the database with the generated credentials
mysql -uv-token-writer-x43s0MOPjCEl6zvkb -pnRG7iHGRC6-dVhrU5uVh -h localhost

mysql» create database globomanticsdb;
Query OK, 1 row affected (0.04 sec)

mysql> create database globomanticsdb; 

mysql> show databases;
+-------—---------—--+
| Database           |
+-------—---------—--+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+-------—---------—--+
4 rows in set (0.00 sec)

show databases;
```

![Alt Image Text](../images/vt5_1_7.png "Body image")



## Working with the SSH Secrets Engine


### Why would end-users request OTPs?

* Mission-critical access
* Minimal amount of time for credentials validity


### Vault OTP Dynamics

![Alt Image Text](../images/vt5_1_8.png "Body image")

1. Enable ssh secrets engine
2. Create a role to connect to SSH servers
3. **Install Vault SSH Helper**
4. **Reconfigure SSHD to use helper**
5. **Test Helper to Vault link**
6. Generate OT dynamically
7. Present OTP to SSH Server
8. Validate OTP
9. Authenticate and revoke OTP

### **SSH Secrets Engine**

**The Vault SSH secrets engine provides secure authentication and authorization for access to machines via the SSH protocol**. 

The Vault SSH secrets engine helps manage access to machine infrastructure, providing several ways to issue
SSH credentials.

The Vault SSH secrets engine supports the following modes. Each mode is individually documented on its own page

* Signed SSH Certificates
* One-time SSH Passwords
* Dynamic SSH Keys (DEPRECATED)

Vault can create a one-time password (OTP) for SSH authentication on a network every time a client wants to SSH into a remote host using a helper command on the remote host to perform verification.

![Alt Image Text](../images/vt5_1_9.png "Body image")

### Why would end-users request OTPs?

* Mission-critical access
* Minimal amount of time for credentials validity


### Enable SSH integrate with Vault

	
```
#!/bin/bash

# The instructions below were tested on Ubuntu 20.04
wget https://releases.hashicorp.com/vault-ssh-helper/0.2.1/vault-ssh-helper_0.2.1_linux_386.zip
unzip vault-ssh-helper_0.2.1_linux_386.zip
mv vault-ssh-helper /usr/local/bin
export VAULT_ADDR=http://localhost:8200/
rm vault-ssh-helper_0.2.1_linux_386.zip
```


### Enable vault-ssh secret

```
$ vault secrets enable ssh
Success! Enabled the ssh secrets engine at: ssh/
```

### ON UBUNTU



```
# .bashrc
export VAULT_ADDR=http://localhost:8200
```

* **`install_vault_ssh_helper.sh`**

```
# Allow you terminal (or your IDE, like VS Code, if you prefer) full disk access - see the screenshot: enable_full_disk_access_on_mac.png

# Enable/disable SSHD on Mac by executing
sudo systemsetup -setremotelogin on

# Execute install_vault_ssh_helper.sh as root
sudo -s
chmod +x install_vault_ssh_helper.sh 

./install_vault_ssh_helper.sh 
```

![Alt Image Text](../images/vt5_1_10.png "Body image")



```
#ls -l /usr/local/bin/

-rwxr-xr-x1 root root 8383402 Dec 15 2020 vault-ssh-helper
```




```
# Install SSHD
apt-get update && apt-get install -y openssh-server

# Modify SSH Config
auth requisite pam_exec.so quiet expose_authtok log=/tmp/vaultsshhelper.log /usr/local/bin/vault-ssh-helper -dev -config=/etc/vault-ssh-helper.d/config.hcl
auth optional pam_unix.so not_set_pass use_first_pass nodelay
```

**`config.hcl`**

```
vault_addr = "http://localhost:8200"
tls_skip_verify = true
# ca_cert = "<PEM_ENCODED_CA_CERT>"
ssh_mount_point = "ssh"
# namespace = "my_namespace"
allowed_roles = "*"
```

```
systemctl restart ssh.service

systemctl status ssh.service
```

![Alt Image Text](../images/vt5_1_11.png "Body image")


```
# Create user that will connect to Vault server
adduser vsshuser

# Create vaultadmin role; vsshuser is the username we will specify when connecting to the ssh server
# cidr_list=0.0.0.0/0,0.0.0.0/0 - this range specifies we can connect from anywhere to the ssh server
vault write ssh/roles/vaultadmin \
    key_type=otp \
    default_user=vsshuser cidr_list=0.0.0.0/0,0.0.0.0/0
```


```
cd ../conf/
# Create config director for the Vault SSH Helper and place the its config file in it
sudo mkdir /etc/vault-ssh-helper.d/
sudo cp -p vault-ssh-helper.hcl

# Test Vault SSH Helper
$ vault-ssh-helper -dev -verify-only -config=/etc/vault-ssh-helper.d/config.hcl
2021/09/15 12:49:11 ==> WARNING: Dev mode is enabled!
2021/09/15 12:49:11 (INFO] using SSH mount point: ssh
2021/09/15 12:49:11 (INF0] using namespace:
2021/09/15 12:49:11 (INFO] vault-ssh-helper verification successful!
```

```
$ ifconfig
inet 10.0.2.15| netmask 255.255.255.0 broadcast 10.0.2.255
```

![Alt Image Text](../images/vt5_1_12.png "Body image")


```
# Generate one-time password (OTP)
vault write ssh/creds/vaultadmin ip=10.0.2.15

Key						Value
lease_id    		ssh/creds/vaultadmin/dU1IHrGwtdaUP17p8LzNMOID
lease_duration    768h
lease_renewable	false
ip				10.0.2.15
key			05dbd824-b832-5454-ac87-7b88f2bc6abf
key_type	otp
port			22
username	vsshuser
```

```
ssh vsshuser@127.0.0.1 -p 2222

root@jaguargs:/home/gsmith/vault/conf#cat/tmp/vaultsshhelper.log
wed Sep 15 15:12:23 Z0ZT
2021/09/15 15:12:23 ==> WARNING: Dev mode is enabled!
2021/09/15 15:12:23 (INFO] using SSH mount point: ssh
2021/09/15 15:12:23 (INFO] using namespace:
2021/09/15 15:12:23 [INFO] vsshuser@10.0.2.15 authenticated!
root@jaguargs:/home/gsmith/vault/conf#ls-l/tmp/vaultsshhelper.log
-rw-r..r.- 1 root root 243 Sep 15 15:12 /tmp/vaultsshhelper.log
root@jaguargs:/home/gsmith/vault/conf# usermod -aG sudo username
usermod: user
"username' does not exist

# usermod -aG sudo vsshuser

# su - vsshuser
To run a command as administrator (user "root"), use "sudo <command>"
See "man sudo root" for details.

$ sudo ls -la /root

$ exit
logout
```

```
ssh vsshuser@127.0.0.1 -p 2222
vsshuser@127.0.0.1's password:
Permission denied, please try again.
vsshuser@127.0.0.1's password:
Permission denied, please try again.
vsshuser@127.0.0.1's password:
```

**Create new one with sudo permisssion**

```
vault write ssh/creds/vaultadminip=10.0.2.15

# After you connect successfully, log out, and try connecting again using the same passowrd
# That will fail because you alerady used the password once
# Enable port forwarding in VirtualBox - VM > Settings > Network  > Port Forwarding - open port 22 on 127.0.1.1 
# and that will forward traffic to port 22 of 10.0.2.15 - internal address of your VM
# OTP: 05dbd824-b832-5454-ac87-7b88f2bc6abf
ssh vsshuser@localhost -p 2222



cat /tmp/vaultsshhelper. log
*** Fri Sep 17 01:03:36 2021
2021/09/17 01:03:36 ==> WARNING: Dev mode is enabled!
2021/09/17 01:03:36 (INFO] using SSH mount point: ssh
2021/09/17 01:03:36 (INFO] using namespace:
2021/09/17 01:03:36 [INFO] vsshuser@10.0.2.15 authenticated!
*** Fri Sep 17 01:22:00 2021
2021/09/17 01:22:00 ==> WARNING: Dev mode is enabled!
2021/09/17 01:22:00 [INFO] using SSH mount point: ssh
2021/09/17 01:22:00 [INFO] using namespace:
2021/09/17 01:22:00 [ERROR]: Error making API request.
URL: PUT http: //10.0.2.2:8200/vl/ssh/verify
Code: 400. Errors:
* OTP not found
*** Fri Sep 17 01:22:03 2021
2021/09/17 01:22:03 ==> WARNING: Dev mode is enabled!
2021/09/17 01:22:03 (INFO] using SSH mount point: ssh
2021/09/17 01:22:03 (INFO] using namespace:
2021/09/17 01:22:03 [ERROR]: Error making API request.
URL: PUT http://10.0.2.2:8200/v1/ssh/verify
Code: 400. Errors:
* OTP not found
*** Fri Sep 17 01:24:40 2021
2021/09/17 01:24:40 ==> WARNING: Dev mode is enabled!
2021/09/17 01:24:40 [INFO] using SSH mount point: ssh
2021/09/17 01:24:40 (INFO] using namespace:
2021/09/17 01:24:40 (INFO] vsshuser@10.0.2.15 authenticated!

```


## Working with the AWS Secret Engine


First, we'll create a Lambda function on AWS and expose it via the AWS API Gateway. 

Then we'll configure Vault to access our AWS account as root. 

Create a role for our Vault users and map it to an AWS IAM policy, which is nothing more than a set of permissions allowing them to take specific actions like accessing and modifying resources.

### Demo

Using the Database Secrets Engine

* Generate Root and Intermediate CAs 
* Create a role that allows for managing subdomains 
* Request and generate certificates 
* Revoke and remove certificates


### Certificate Revocation Lists (CRLs)

* Generate Credentials  as dynamic secrets
* Verify certificate is NOT on CRLs

![Alt Image Text](../images/vt5_1_13.png "Body image")

![Alt Image Text](../images/vt5_1_14.png "Body image")

**AWS Root Keys**

* Don't share them with anyone
* Vault uses them to create new secrets

### Configrue Lambda trigger

**Trigger configuration**

* Create an API
* REST API
* IAM

![Alt Image Text](../images/vt5_1_15.png "Body image")

![Alt Image Text](../images/vt5_1_16.png "Body image")

![Alt Image Text](../images/vt5_1_17.png "Body image")


```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "execute-api:Invoke",
            "Resource": "arn:aws:execute-api:us-east-2:239136941756:*/*/*/*"
        }
    ]
}
```

```
# Enable AWS secrets engine
vault secrets enable aws

# Configure Vaut’s root access
vault write aws/config/root \
    access_key=A*6 \
    secret_key=n*D \
    region=us-east-2
```

### AWS API Standards

```
# Create role in Vault and link it to an AWS policy
# this role will allow users/apps to call the Lambda function via the API gateway
vault write aws/roles/dev-role \
    credential_type=iam_user \
    default_ttl=5m
    policy_document=-<<EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DevRoleInlinePolicy1",
            "Effect": "Allow",
            "Action": "apigateway:*",
            "Resource": "*"
        }
    ]
}
EOF
```

```
vault read aws/creds/dev-role
Key                Value
---                -----
lease_id           aws/creds/dev-role/PHsJqJuWMfDMP0gBoYfZLDqV
lease_duration     768h
lease_renewable    true
access_key         AKIATPLNME26DOZTWH7S
secret_key         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
security_token     <nil>
```

**Vault generated IAM user**

```
vault-token-dev-role-1632044346-8561
```

![Alt Image Text](../images/vt5_1_19.png "Body image")

![Alt Image Text](../images/vt5_1_20.png "Body image")

![Alt Image Text](../images/vt5_1_18.png "Body image")


### **Managing Leases from Dynamic Secrets**

All dynamic secrets in Vault come with a lease. 

And so, there are three main cases surrounding the lease and how it **governs a secrets lifecycle. Secrets expire, are renewed, or can be revoked**


* **Control lease expiration**


```
vault secrets tune -default-lease-ttl=5m aws/
vault secrets tune -default-lease-ttl=5m -max-lease-ttl=20m aws/
```

* **Renew lease** 

```
# Renew lease for 600 sec, or 10 min - within max_lease_ttl of 20 min - see above
vault lease renew -increment=600 my-lease-id
vault lease lookup aws/creds/dev-role/fZWeCwRT8l9xbRxrDoEpDRTe
```

* **Revoke credentials**

```
# Revoke credentials
vault lease revoke aws/creds/dev-role/CdTx14Wl0m1kyUI6luKlEEsl
```

