# **L1 Managing the Encryption Key**

## **1 Introduction**

### Four modules

**Managing the encryption key**

* Highlight Security concerns

**Managing the Storage backend**

* Discuss the operational complexities

**Managing Vault during an incident**

* Troubleshoot the Vault server

**Managing plugins and server upgrades**

* Demonstrate server upgrade


## **2 Installing Vault**

```
# Now that we have a K8s cluster to work with, let's get Vault deployed with Helm
export SECRET_NAME=vault-tls
export NAMESPACE=vault
export certificate_cn=""

# First we will add the Helm repo for Vault
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update hashicorp

# Add a namespace for the Vault cluster
kubectl create namespace ${NAMESPACE}

# Create a secret with the Vault certificate info
kubectl create secret generic vault-tls \
  --namespace ${NAMESPACE} \
  --from-file=vault.key=./certs/vault.key \
  --from-file=vault.crt=./certs/vault.crt
```

* **`vault.key`**

```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAp+l+jV5wiDRbwOaTWWGBPW/fLga2ifMlkD2JMyCeoe/odPQo
7n17gI3GD62UfZJZGJBAYQHLMiGbbS863ftQd+EZ5L54tkZC8aHxyokAN/zwGX/q
M8dULsy/6sDfP2qF3TlccIKTjkEemRsGt2hQGbgyIQpljchwJhHSOrBRJf9UDu+R
COvuK8RBd9eIrvVKu1OmQXrhOQQnjfCE+u/aM9kWCjUR2I/ZdLxsVIwjcDY8al7q
/NLZ+j+ZEseHHrL487umwTWO0wl/ixcJQqbJSxCblMaiTlt9nWl5cuHzknDCk/Fz
SlOOdBvmWRr42+JC9EX27furk6R4lGjmcWRu9QIDAQABAoIBAET3bJlYHtRy3PTg
P8lIeTLozjwzWapTiiwyejXbwgw1Y78VwfIi7CUjFiS+YO0qvRmPtkGh4PnYP+Sa
r40ZejWi6WXArgf/1+MzZwKPPt3AMEXo+iMGThdi1bjwO0Hq2MzCkKtNlpmpOY4d
Qab3W18U/tH/WTLhDt80FqswXEREbsth3B6v4MEsnBzDrzO+f0J4VobjPvVKfb38
NdUxfa9c+C6MFQ/pbA+OR5fOgiTUBpXQkIP9B7IUbBcZSQWH1AdlzhzvLCUQC9uU
p1ULBcibvsyKuJfzaIPeN5OitECKjJP1/OPkwIiyhQmlZniLvkoMbEqoJxA86Yeo
R5M39UECgYEAz6mlgfPDIOXzeEoj74/Jz0Lb/s0b2rH1ImyzwAh6CR/mZr5dCja9
nci+eoO9EMyf4VPt+gmsy0Rg5VdF3g8WP5GwXONlTPdgPu1j5oNFKy9agcG+Je/r
RYCpHXUhNCtJZ32JcOK7+/pWTqjx5XwGe1wEp1DVgAgMrJX2xnj68KkCgYEAzv8p
KGTgEmcvGPA6flO1PWpvFdbX+tMgt/BuPnULQRU3Vbt0eqGqGYBWDoRh84viyWPw
kTG2UE1oLqsAn2QEo3p8wHkCq8NjjTws9G9blIZ6qq75ctomL6LNc20Y/GP9GaHh
QinMELizkDr9FGIQGQZDpbXfL/kwzUNNeq9bn20CgYEAuuLOFJmv7heEeakfliUt
Vd2x5ynn+3VUS1rQme4HBdcznxuK1/pTw4A6DEArEsdUy5ChBvlWDMjI+x0Dv8iM
GUGgGrh7Hj0y3O7/jSBuS2MebNBTtAirD62uhNg4vZ6HPR/5ZSJoU2kD616X9BMe
Mj6p4XoZ6lNUNK3xnMSb8aECgYATfa1Hc3Ax62tg4sXvPTyUouAA43EE2jp7d3U6
BlVf9Hp4ZSVLostciVwRE+ARSbu6EOIHGfI7z34e3tU5LPbO2erDSOG6gLRwCRmG
jrHF43/LcJxmdyofVCoaU0yF/3hGoJqQW1UuweWOkC9iavVvFcrq4Jw6bTFuMocq
JTbwZQKBgQCSr85uE9+oBeXW0tWhoNhuuxWqlR+AEFTp+OAoaG2nthmjx1Nl/kPG
0T1/UevLL1Np6EO7dLLgo2kSNCxKgfG9IKv3sA6MDfhxYWyYfweiklxTtpCrCO+/
ieCO0qespJ5x190+9kqVf8SMqxxcScgfjQjaUrPSnH6svUmqDik5OQ==
-----END RSA PRIVATE KEY-----
```

* `vault.crt`

```
-----BEGIN CERTIFICATE-----
MIIC9jCCAd4CCQCXVSNk7loeKTANBgkqhkiG9w0BAQUFADA9MRUwEwYDVQQKDAxz
eXN0ZW06bm9kZXMxJDAiBgNVBAMMG3N5c3RlbTpub2RlOnZhdWx0LnZhdWx0LnN2
YzAeFw0yMjA3MDEwODUyMjRaFw0yMzA3MDEwODUyMjRaMD0xFTATBgNVBAoMDHN5
c3RlbTpub2RlczEkMCIGA1UEAwwbc3lzdGVtOm5vZGU6dmF1bHQudmF1bHQuc3Zj
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAp+l+jV5wiDRbwOaTWWGB
PW/fLga2ifMlkD2JMyCeoe/odPQo7n17gI3GD62UfZJZGJBAYQHLMiGbbS863ftQ
d+EZ5L54tkZC8aHxyokAN/zwGX/qM8dULsy/6sDfP2qF3TlccIKTjkEemRsGt2hQ
GbgyIQpljchwJhHSOrBRJf9UDu+RCOvuK8RBd9eIrvVKu1OmQXrhOQQnjfCE+u/a
M9kWCjUR2I/ZdLxsVIwjcDY8al7q/NLZ+j+ZEseHHrL487umwTWO0wl/ixcJQqbJ
SxCblMaiTlt9nWl5cuHzknDCk/FzSlOOdBvmWRr42+JC9EX27furk6R4lGjmcWRu
9QIDAQABMA0GCSqGSIb3DQEBBQUAA4IBAQBmogQGNT+p429ltJD/fsBRd58r0hSS
xZPLObKehl6jZYb4Dp3Up4Nh0ZkykzJeA/9bHqqaTzf77+/2uxmYwNCdxsLz4Fff
zbEfeyVDGImHiKZiIG814sjWUkt5FD4m5w4oMDtH9Y4j48oXdMDVCCSGgx2qH2p1
tmBiSa5MHmK1QI8d+k4xkqNxowExeVdMcMRAa2ey0mdp3fuCDSPltb7rLPtl0IaH
MtZZSyfV5rjbfnjBs4uXrIJIjsraqWJz+MdoaanZs/CiMUgzZhb8iOaFPZYyXMIz
709mgDIqmtWPlPVXetixuMG5KXP5yoaM0YnjvHfwz0mGkH130EuIBjuJ
-----END CERTIFICATE-----
```

### Install Consul

```
# Deploy Consul to provide storage for Vault
helm install consul hashicorp/consul --namespace vault
```
### Install Vault

```
# Deploy Vault cluster to K8s using helm
helm install vault hashicorp/vault \
  --namespace vault \
  --values values.yaml
```

* **`values.yaml`**

```
# Vault Helm Chart Value Overrides
global:  
  enabled: true  
  tlsDisable: false

server:

  readinessProbe:
    enabled: true
    path: "/v1/sys/health?standbyok=true&sealedcode=204&uninitcode=204"
  livenessProbe:
    enabled: true
    path: "/v1/sys/health?standbyok=true"
    initialDelaySeconds: 60

  extraEnvironmentVars:
    VAULT_CACERT: /vault/userconfig/vault-tls/vault.ca
    VAULT_SKIP_VERIFY: true

  extraVolumes:
    - type: secret
      name: vault-tls

  standalone:
    enabled: "true"

    # config is a raw string of default configuration when using a Stateful
    # deployment. Default is to use a PersistentVolumeClaim mounted at /vault/data
    # and store data there. This is only used when using a Replica count of 1, and
    # using a stateful set. This should be HCL.

    # Note: Configuration files are stored in ConfigMaps so sensitive data
    # such as passwords should be either mounted through extraSecretEnvironmentVars
            # tls_client_ca_file = "/vault/userconfig/vault-tls/vault.ca"
    # or through a Kube secret.  For more information see:
    # https://www.vaultproject.io/docs/platform/k8s/helm/run#protecting-sensitive-vault-configurations
    config: |
      
      listener "tcp" {
        address = "[::]:8200"
        cluster_address = "[::]:8201"
        tls_cert_file = "/vault/userconfig/vault-tls/vault.crt"
        tls_key_file  = "/vault/userconfig/vault-tls/vault.key"

      }


      storage "consul" {
        path = "vault"
        address = "consul-consul-server:8500"
      }
      ui = true
      log_level = "Debug"


  service:
    type: LoadBalancer
    annotations: |
      "service.beta.kubernetes.io/aws-load-balancer-name": "vault-server"
```

```
# We can monitor the install by doing a watch on the namespace
kubectl get pods -n vault -w

NAME	READY	STATUS	RESTART	SAGE
consul-consul-client-7zx62	1/1 Running 0 17m
consul-consul-client-crr7j	1/1 Running 0 17m
consul-consul-client-djppb	1/1 Running 0 17m
consul-consul-server	    1/1 Running 0 17m
consul-consul-server-1	1/1	Running 0 17m
consul-consul-server-2	1/1	Running 0 17m
vault-agent-injector-6bf6cdb945-6g65z 1/1 Running 0 15m
Vault-1/1		Running	1/1 1（10sago）90S
```

```
# Once they are all Running, we're in good shape
# Now get the LoadBalancer IP address for the server
kubectl get service vault -n vault

NAME TYPE CLUSTER-IP EXTERNAL-IP PORT (S) AGE
vault loadBalancer 19. 199.49. 175 466846f95751a-1973715929.us-east-1.elb.amazonaws.com
8200:32339/TCP,820130339/TCP 16m
```

```
# The address for the Vault server will be the dns label 
# plus the Azure region cloudapp.azure.com 
# Ex. vaultf-9f8a7a6.eastus.cloudapp.azure.com
# You'll need to add a CNAME entry for this to your public DNS
# Ex. vault-aks.globomantics.xyz

export VAULT_ADDR="https://${certificate_cn}:8200"
```


```
vault status -tls-skip-verify
key  		vaule
---   		----
seal Type	 shamir
···
```

## 2 Unsealing the Vault

Vault server is sealed. You need to unseal it. You can unseal the Vault server in three different ways.

* **Manual unsealing**

2 or more operators enter encryption key manually

* **Auto unsealing**

A cloud key management service (KMS) is used

* **Transit unsealing**

One vault server is used to unseal another

```
## Unsealing command 
vault operator init -tls-skip-verify -key-shares=3 -key-threshold=2

Unseal Key 1: munsuu2SP6nXp1Fgk4PcIi+SA2DrAT8ACelvxtBn9BW7
Unseal Key 2: WAqRCuM4nsG®J+6pDPxovAhW568Q06EVnoxaNISMo9of
Unseal Key 3: ZxgC1G7T1hXDQkI1copnPRFdPgfShSBSWc/PV+AnvnJ5

Initial Root Token: hvs.0Qy1NYRSVD8VQsU8IUDLstQJ

Vault initialized with 3 key shares and a key threshold of 2. Please securely
distribute the key shares printed above. When the Vault is re-sealed.
restarted, or stopped, you must supply at least 2 of these keys to unseal it
before it can start servicing requests.
Vault does not store the generated root key. Without at least 2 keys to
reconstruct the root key, Vault will remain permanently sealed!
It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. 

See "vault operator rekey" for more information.
```

### Manual unsealing method with GPG

You can use the PGP, GnuPG, or keybase keys. 

There is an option in the vault init command in which you can pass the GPG keys of the operators who are handling the unsealing keys. 

**The Vault will then return the unsealing keys encrypted with those GPG keys.**

You can then share those unsealing keys over chat or email to the respective operator. 

![Alt Image Text](../images/chap6_1_1.png "Body image")

```
### import pgp keys
gpg --import dev-x.asc
gpg --import dev-y.asc
gpg --import dev-z.asc
```

* **`dev-x-encoded.asc`**

```
-----BEGIN PGP PUBLIC KEY BLOCK-----

mDMEYr7vURYJKwYBBAHaRw8BAQdAbs7pgAejCRBNnJu/ljyFBvKvka7ipnRof+ti
fgA6h+e0G2Rldi14IDxkZXYteEB2YXVsdGRlbW8uY29tPoiZBBMWCgBBFiEEx+DS
/UcaKsnx22UgvoGy1kjA3ywFAmK+71ECGwMFCQPCZwAFCwkIBwICIgIGFQoJCAsC
BBYCAwECHgcCF4AACgkQvoGy1kjA3ywnYgEAk+OYqhhGiSv6311WzCRV/IPZldgO
f0bk+wb7YHw6Z2MA/idvU4EAV5qvG3nvzX1BkMZU+RIfkluMR03aG6JoexYFuDgE
Yr7vURIKKwYBBAGXVQEFAQEHQBwBNRy1OdcfIrjcVFswZqqZm5go1BLammm+yazS
+SNEAwEIB4h+BBgWCgAmFiEEx+DS/UcaKsnx22UgvoGy1kjA3ywFAmK+71ECGwwF
CQPCZwAACgkQvoGy1kjA3yztzwD/azc7Mrg4DkvFk2dsGeCm9fFCjNOAK5KYDpC5
zXuFp3wBAM61CBVuMVmu+xGrfEvka4cLFy83W8CwzRd2Pa01/5sH
=LqfI
-----END PGP PUBLIC KEY BLOCK-----
```

* **`dev-y-encoded.asc`**

```
-----BEGIN PGP PUBLIC KEY BLOCK-----

mDMEYr70xBYJKwYBBAHaRw8BAQdARciLwv4WRskXRrNpm5yBDQlOZnribYn9egut
D0jO+OG0G2Rldi15IDxkZXYteUB2YXVsdGRlbW8uY29tPoiZBBMWCgBBFiEEkIMR
wMAjc0Rklkz+HwT+dJXNVcwFAmK+9MQCGwMFCQPCZwAFCwkIBwICIgIGFQoJCAsC
BBYCAwECHgcCF4AACgkQHwT+dJXNVcyzaQEA5jei8Svjqzs4wU/+VpVisgTwgse6
v1+wuxLkKJFfSFcA/29esVLpbc+S8uXlMj2dEbd3yKg7prbqale3DK24bEoMuDgE
Yr70xBIKKwYBBAGXVQEFAQEHQBYwyIY0D7Pe2F18yn7CNYnBd9uqtN3yReXTtI1t
ESsyAwEIB4h+BBgWCgAmFiEEkIMRwMAjc0Rklkz+HwT+dJXNVcwFAmK+9MQCGwwF
CQPCZwAACgkQHwT+dJXNVcxHCwEAx8M/eUWAr3kmz3YSsAl1pR26o7tBLiuahtVd
nq38ntkBALmrtPzD9hSiEVR+hJaSAgN01KXmwEsvvrQSHXhHpGQM
=WIul
-----END PGP PUBLIC KEY BLOCK-----
```

* `dev-z-encoded.asc`

```
-----BEGIN PGP PUBLIC KEY BLOCK-----

mDMEYr703RYJKwYBBAHaRw8BAQdA9QV2avIfeVHy72z6h2rIVlRePBiuyjdaLh4H
nRTL8xu0G2Rldi16IDxkZXYtekB2YXVsdGRlbW8uY29tPoiZBBMWCgBBFiEEjVBp
XY3YIsd3YD2KMQtnpoP/bhwFAmK+9N0CGwMFCQPCZwAFCwkIBwICIgIGFQoJCAsC
BBYCAwECHgcCF4AACgkQMQtnpoP/bhzP9AEAmiIoe09CMqyD4HztKhjbrnMG5C/w
xn3QM6gycJsRI94BANIttzCnAJTymNwkm+PJtHemI5sVq17DZytfCuBTh6kLuDgE
Yr703RIKKwYBBAGXVQEFAQEHQNETh+oi8848nudzbicVPBfA0FcUqtfU5S3oFDer
2m12AwEIB4h+BBgWCgAmFiEEjVBpXY3YIsd3YD2KMQtnpoP/bhwFAmK+9N0CGwwF
CQPCZwAACgkQMQtnpoP/bhzBhwEAk2KC8A/+wesH5NImZvlngee2gDM+WtvFnGyJ
s7MaHDIA/AsFDQenANjqtUuek/y5nzBlMcZAQ5u3Nb12Fb4glNIC
=1hNn
-----END PGP PUBLIC KEY BLOCK-----
```

```
$ gpg --list-keys
```

![Alt Image Text](../images/chap6_1_2.png "Body image")

```
### export public keys to disk with base64 encoding

gpg --export -a dev-x > dev-x-encoded.asc
gpg --export -a dev-y > dev-y-encoded.asc
gpg --export -a dev-z > dev-z-encoded.asc
```


```
vault operator init -tls-skip-verify -key-shares=3 -key-threshold=2 -pgp-keys="dev-x-encoded.asc,dev-y-encoded.asc,dev-z-encoded.asc

* Unseal Key 1: wUADBdRFLivxYIWSAQdA7INCsya24F/EDByraPMoE@FVumEZ6y68FuFFkHBN@jEgdRyrWyeDM5yr+/ekdRxaS8mCFcieTNJdfX2CqrYxM2LS4AHkp9PKe02KARpgS+KQnelcH+GAfuC64N7hsezgTOJAAwsY40rmj9MOuY9zD3h7srA3GsPyXBDMjIKfdkPK@wp45y00J6IQzF50WIQ4P3VHhPuU6XUeSEpLOvuJ3z9d770FfQ6GeDB4U464FLkbXaP@+i1/3kkfKmhQ6aeReJltugC4ZF5AA==
* Unseal Key 2: wU4Dn2PrEmM8HCASAQdAcEWojkCHyI27mxW9Hq+FK9rf0e6V45m5TcG+Wkqp9ykqyh9Lv43ZJpxNDLZCOExmSduxR8yIsI2mfToEs80gDrvS4AHkYy5+TuS3rKLLPbmYgJQ5r+Fq1eD24Drh4ILgGuI59TdH4Bfm4heNqYY/aB/30pkmnHPegkJjuOZPf52Ptg2nFSDAwhPccbnx+LUg1JYif015SJVRYTfTgX5iN86K6FGkWwFnuBK4RRU4GTksiNTZ7ymn611pSU117pij+J05cfj4Wf+AA==
* Unseal Key 3: wU4DkFP01xIC1Z0SAQdA+7x18qF9JrGw331Rx615kpiW1Mi4Wb+WX/DhuweT91MqiubgefE4YkRa1G7fIrts1ofSVII607fPYR/VRnrvWKzS4AHktf@JAQ8hMqQpo/BOSPkWZOHI50AZ4Nzh5nPgx0L97giN4AnmfvEAISIXtd8ZDi3eJNiOF+Xf8VMmG5vtzrRkv17hog7yNKtwkfJ49DRjT5BikKP6NG5b3PHGjTOzSAmY6JLZZ+Bp4aFi4D3k2nXWgrwnPqYE6QZ5tg/OhuI1xC864Z24AA==

Initial Root Token: hvs.NIWJmUZJm4kno8tcoHmeT67X

Vault initialized with 3 key shares and a key threshold of 2. Please securely
distribute the key shares printed above. When the Vault is re-sealed,restarted, or stopped, you must supply at least 2 of these keys to unseal it before it can start servicing requests, Vault does not store the generated root key. Without at least 2 keys to

Vault does not store the generated root key. Without at least 2 keys to
```

* Root Token: hvs.NIWJmUZJm4kno8tcoHmeT67X

```
$ echo wUADBd... | base64 --decode | gpg -dq

a8373b92f595ece47a9159aada8d4d880b2b4da7c7a64378031ed23a8f418037d2

echo wU4Dn2PrEmM8... | base64 --decode | gpg -dq

382492d0353209ee28bd05dc96f42aee2b62e2c62c1f18579c67ec3053317647fd
```

```
$ vault operator unseal -tls-skip-verify
....
Unseal Progress 1/2

$ vault operator unseal -tls-skip-verify
```

## Rekeying Rotation

This vault server will remain unsealed until the vault server is restarted, there is an error at the storage layer of the vault server, or the vault server is sealed manually or via the API in case of a cyberattack. 

### Vault stays unsealed until:

* Server is restarted
* An error at storage layer
* Sealed manually via API

### What if?

* An operator is on vacation or leaving the organisation
* A new team member joins and shares the unsealing responsibilities
* The team wants to increase theshards and threshold of unsealing keys
* Compliance mandates rotating the encryption keys periodically

### Possible Solutions:

* Rekey the unsealing keys
* Rotate the encryption key
* Migrate to auto-unseal

> **Transfer the unsealing responsibilities.**

### Explore the rekey and rotation method

* **`dev-x-encoded.asc`**

```
-----BEGIN PGP PUBLIC KEY BLOCK-----

mDMEYr7vURYJKwYBBAHaRw8BAQdAbs7pgAejCRBNnJu/ljyFBvKvka7ipnRof+ti
fgA6h+e0G2Rldi14IDxkZXYteEB2YXVsdGRlbW8uY29tPoiZBBMWCgBBFiEEx+DS
/UcaKsnx22UgvoGy1kjA3ywFAmK+71ECGwMFCQPCZwAFCwkIBwICIgIGFQoJCAsC
BBYCAwECHgcCF4AACgkQvoGy1kjA3ywnYgEAk+OYqhhGiSv6311WzCRV/IPZldgO
f0bk+wb7YHw6Z2MA/idvU4EAV5qvG3nvzX1BkMZU+RIfkluMR03aG6JoexYFuDgE
Yr7vURIKKwYBBAGXVQEFAQEHQBwBNRy1OdcfIrjcVFswZqqZm5go1BLammm+yazS
+SNEAwEIB4h+BBgWCgAmFiEEx+DS/UcaKsnx22UgvoGy1kjA3ywFAmK+71ECGwwF
CQPCZwAACgkQvoGy1kjA3yztzwD/azc7Mrg4DkvFk2dsGeCm9fFCjNOAK5KYDpC5
zXuFp3wBAM61CBVuMVmu+xGrfEvka4cLFy83W8CwzRd2Pa01/5sH
=LqfI
-----END PGP PUBLIC KEY BLOCK-----
```

* `dev-y-encoded.asc`

```
-----BEGIN PGP PUBLIC KEY BLOCK-----

mDMEYr70xBYJKwYBBAHaRw8BAQdARciLwv4WRskXRrNpm5yBDQlOZnribYn9egut
D0jO+OG0G2Rldi15IDxkZXYteUB2YXVsdGRlbW8uY29tPoiZBBMWCgBBFiEEkIMR
wMAjc0Rklkz+HwT+dJXNVcwFAmK+9MQCGwMFCQPCZwAFCwkIBwICIgIGFQoJCAsC
BBYCAwECHgcCF4AACgkQHwT+dJXNVcyzaQEA5jei8Svjqzs4wU/+VpVisgTwgse6
v1+wuxLkKJFfSFcA/29esVLpbc+S8uXlMj2dEbd3yKg7prbqale3DK24bEoMuDgE
Yr70xBIKKwYBBAGXVQEFAQEHQBYwyIY0D7Pe2F18yn7CNYnBd9uqtN3yReXTtI1t
ESsyAwEIB4h+BBgWCgAmFiEEkIMRwMAjc0Rklkz+HwT+dJXNVcwFAmK+9MQCGwwF
CQPCZwAACgkQHwT+dJXNVcxHCwEAx8M/eUWAr3kmz3YSsAl1pR26o7tBLiuahtVd
nq38ntkBALmrtPzD9hSiEVR+hJaSAgN01KXmwEsvvrQSHXhHpGQM
=WIul
-----END PGP PUBLIC KEY BLOCK-----
```

* **`dev-z-encoded.asc`**

```
-----BEGIN PGP PUBLIC KEY BLOCK-----

mDMEYr703RYJKwYBBAHaRw8BAQdA9QV2avIfeVHy72z6h2rIVlRePBiuyjdaLh4H
nRTL8xu0G2Rldi16IDxkZXYtekB2YXVsdGRlbW8uY29tPoiZBBMWCgBBFiEEjVBp
XY3YIsd3YD2KMQtnpoP/bhwFAmK+9N0CGwMFCQPCZwAFCwkIBwICIgIGFQoJCAsC
BBYCAwECHgcCF4AACgkQMQtnpoP/bhzP9AEAmiIoe09CMqyD4HztKhjbrnMG5C/w
xn3QM6gycJsRI94BANIttzCnAJTymNwkm+PJtHemI5sVq17DZytfCuBTh6kLuDgE
Yr703RIKKwYBBAGXVQEFAQEHQNETh+oi8848nudzbicVPBfA0FcUqtfU5S3oFDer
2m12AwEIB4h+BBgWCgAmFiEEjVBpXY3YIsd3YD2KMQtnpoP/bhwFAmK+9N0CGwwF
CQPCZwAACgkQMQtnpoP/bhzBhwEAk2KC8A/+wesH5NImZvlngee2gDM+WtvFnGyJ
s7MaHDIA/AsFDQenANjqtUuek/y5nzBlMcZAQ5u3Nb12Fb4glNIC
=1hNn
-----END PGP PUBLIC KEY BLOCK-----
```


```
## start the rekeying
vault operator rekey -tls-skip-verify -init -key-shares=3 -key-threshold=2 -pgp-keys="dev-x-encoded.asc,dev-y-encoded.asc,dev-z-encoded.asc"
```

![Alt Image Text](../images/chap6_1_3.png "Body image")

```
nonce id: d26464d0-1ecb-84fc-25f1-780bd9caf3a
```

```
## Enter the unseal keys
vault operator rekey -tls-skip-verify


vault operator rekey -tls-skip-verify
Rekey operation nonce: d26464d0-lecb-84fc-25f1-7b80bd9caf3a
```

![Alt Image Text](../images/chap6_1_4.png "Body image")

```
### unseal keys
Key 1 fingerprint: c7e0d2f471a2ac9f1db6520be81b2d648c0df2c; value: wU4DBdRFLivxYlwSAQdAqVEjrKh
Key 2 fingerprint: 908311c0c023734464964cfelf04fe7495cd55cc; value: wU4Dn2PrEmM8HCASAQdAsT1UVfH
Key 3 fingerprint: 8d50695d8dd822c777603d8a310b67a683ff6e1c; value: wU4DkFPOxIC1ZOSAQdAAZoMesq
```

### Rekeying

> Online process, so no downtime

![Alt Image Text](../images/chap6_1_5.png "Body image")

**Rotating**

```
vault operator rekey -tls-skip-verify

Error rotating key: Error making API request.
URL: POST https: //ab3302c5d721442a389c0e666b55e046-1966045790.us-east-1.elb.amazonaws.com:8200/v1/sys/rotate
Code: 403. Errors:

* permission denied
```

### **Generate a new root token..**

```
vault operator generate-root-tls-skip-verify -init

Nonce.  da76c47a-9821-97a8-4a42-c9530364d5a4
Started true
Progress 0/2
Complete  false
OTP    gOkDVGzGc3a04kAUUxzs5JK1Tg1m
OT Length 28
```

```
vault operator generate-root -tls-skip-verify
Operation nonce: da76c47a-9821-97a8-4a42-c9530364d5a4
```

```
echo wU4Dn2PrE..  | base64 --decode | gpg -dq

382492d0353209ee28bd05dc96f42aee2b62e2c62c1f18579c67ec3053317b47fd
```


```
vault operator generate-root-tls-skip-verify
Operation nonce: da76c47a-9821-97a8-4a42-c9530364d5a4
Unseal Key (will be hidden) :
Nonce		da76c47a-9821-97a8-4a42-c9530364d5a4
Started		true
Progress 	1/2
Complete 	false

vault operator generate-root -tls-skip-verify
Operation nonce:   da76c47a-9821-97a8-4a42-c9530364d5a4
Unseal Key (will be hidden):
Nonce		da76c47a-9821-97a8-4a42-c9530364d5a4
Started	 true
Progress	2/2
Complete	true
Encoded Token DzkYaj5zAigwQVByYgg5BjQuFj1YKyJZYwwCAg
```

```
nonce id: d26464d0-1ecb-84fc-25f1-780bd9caf3a

decoded = DzkYai5zAiaWOVByYgg5BiQuFilYKyJZYwwCAg
```

```
vault operator generate-root -tls-skip-verify -decode=DzkYai5zAiaWOVByYgg5BiQuFilYKyJZYwwCAg -otp= gOkDVGzGc3a04kAUUxzs5JK1Tg1m
```

### set the new token

```
vault login -tls-skip-verify

Key     Value
token   hvs.h4xour1BVcxSaVlJmaih7k30
...
```

### Start the key rotation

```
vault operator rotate -tls-skip-verify

Success! Rotated key
Key Term.     3
Install Time 	02 Jul 22 04:42 UTC
Encryption Count	 2
```

### Automate the rotation

```
vault read -tls-skip-verify sys/rotate/config

Key 				Value
enabled		true
interval		0
max_operations	3865470566
```

```
vault read -tls-skip-verify sys/rotate/config 

vault write -tls-skip-verify sys/rotate/config interval=3600h

vault write -tls-skip-verify sys/rotate/config max_operations=""
```

## **3 Auto Unsealing**

![Alt Image Text](../images/chap6_1_6.png "Body image")

![Alt Image Text](../images/chap6_1_7.png "Body image")

> Attach this to vault's AM role..

```
{
	"Version": "2012-10-17",
	"Statement": [
		{
		  "Effect": "Allow",
		  "Action": [
		  	"kms: Encrypt",
		  	"kms: Decrypt",
		  	"kms: DescribeKey"
		  ],
		  "Resource": "*"
		}
	]
}
```

* **`values.yaml`**

```
# Vault Helm Chart Value Overrides
global:  
  enabled: true  
  tlsDisable: false

server:

  readinessProbe:
    enabled: true
    path: "/v1/sys/health?standbyok=true&sealedcode=204&uninitcode=204"
  livenessProbe:
    enabled: true
    path: "/v1/sys/health?standbyok=true"
    initialDelaySeconds: 60

  extraEnvironmentVars:
    VAULT_CACERT: /vault/userconfig/vault-tls/vault.ca
    VAULT_SKIP_VERIFY: true

  extraVolumes:
    - type: secret
      name: vault-tls

  standalone:
    enabled: "true"

    # config is a raw string of default configuration when using a Stateful
    # deployment. Default is to use a PersistentVolumeClaim mounted at /vault/data
    # and store data there. This is only used when using a Replica count of 1, and
    # using a stateful set. This should be HCL.

    # Note: Configuration files are stored in ConfigMaps so sensitive data
    # such as passwords should be either mounted through extraSecretEnvironmentVars
            # tls_client_ca_file = "/vault/userconfig/vault-tls/vault.ca"
    # or through a Kube secret.  For more information see:
    # https://www.vaultproject.io/docs/platform/k8s/helm/run#protecting-sensitive-vault-configurations
    config: |
      
      listener "tcp" {
        address = "[::]:8200"
        cluster_address = "[::]:8201"
        tls_cert_file = "/vault/userconfig/vault-tls/vault.crt"
        tls_key_file  = "/vault/userconfig/vault-tls/vault.key"

      }
      seal "awskms"
      {
        region     = "us-east-1"
        kms_key_id = "b7576d39-d159-46f4-b84b-33f5b618fce2"
      
      }

      storage "consul" {
        path = "vault"
        address = "consul-consul-server:8500"
      }
      ui = true
      log_level = "Debug"


  service:
    type: LoadBalancer
    annotations: |
      "service.beta.kubernetes.io/aws-load-balancer-name": "vault-server"
```

```
### Stop the vault server
kubectl scale statefulsets -n vault vault --replicas=0

### update vault helm deployment
helm upgrade vault hashicorp/vault \
  --namespace vault \
  --values values.yaml

### Start vault server 
kubectl scale statefulsets -n vault vault --replicas=1
```

```
$ vault status -tls-skip-verify

Key				     Value
---					  ----
Recovery Seal Type		shamir
....
Storage Type			consul
```

### Migrate the seal

```
vault operator unseal -tls-skip-verify -migrate
```

![Alt Image Text](../images/chap6_1_8.png "Body image")

```
### restart vault
kubectl delete pods -n vault vault-0

### check the pods
kubectl get pods -n vault -w
```


## Associating Policies

### Root Token

![Alt Image Text](../images/chap6_1_9.png "Body image")

* **`dev-y-policy.hcl`**

```
# List, create, update, and delete key/value secrets
path "secret/*"
{
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# List, create, update, and delete key/value secrets at kv-v2/ path
path "kv-v2/*"
{
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}
path "kv/*"
{
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}
# Deny access to developer z's secret 
path "secret/data/dev-z-secret" 
{
    capabilities = ["deny"]
}

# Manage secrets engines
path "sys/mounts/*"
{
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# Create and manage ACL policies
path "sys/policies/acl/*"
{
  capabilities = ["create", "read", "update", "list"]
}
```

* **`dev-z-policy.hcl`**

```
# List, create, update, and delete key/value secrets
path "secret/*"
{
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# Deny access to developer z's secret 
path "secret/data/dev-y-secret" 
{
    capabilities = ["deny"]
}

# Manage secrets engines
path "sys/mounts/*"
{
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# Create and manage ACL policies
path "sys/policies/acl/*"
{
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}
```

### create or update  a policy


```
vault policy write -tls-skip-verify dev-y-policy policy/dev-y-policy.hcl

Success! Uploaded policy: dev-y-policy

vault policy write -tls-skip-verify dev-z-policy policy/dev-z-policy.hcl

Success! Uploaded policy: dev-z-policy
```

```
## List all policies
vault policy list -tls-skip-verify

default
dev-y-policy
root
```

### Read the policy

```
vault policy read -tls-skip-verify dev-y-policy 
```

### Generate a token for the policy

```
vault token create -tls-skip-verify -format=json -policy="dev-y-policy" -ttl 1h -use-limit=2

....
hvs.xxxx
```

### Authenticate with that token

```
export VAULT_TOKEN=""
```

**or**

```
vault login -tls-skip-verify
```

```
$ vault kv list -tls-skip-verify secret/Keys

dev-y-secret
dev-z-secret
my-secret
```

```
vault kv get -tls-skip-verify secret/my-secret
```

![Alt Image Text](../images/chap6_1_10.png "Body image")

```
vault kv get -tls-skip-verify secret/dev-z-secret
```

![Alt Image Text](../images/chap6_1_11.png "Body image")

### Root Token

* Can do anything
* Never expires
* Should be revoked after initial setup
* Should be generated on the fly