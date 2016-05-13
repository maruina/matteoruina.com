+++
date = "2016-04-21T09:20:18+01:00"
draft = true
title = "Consul handbook"
tags = ["consul", "hashicorp"]

+++
## Security
### ACL
The ACL is Capability-based, relying on tokens to which fine grained rules can be applied. It is very similar to AWS IAM in many ways.  

Every token has an ID, name, type, and rule set. The ID is a randomly generated UUID, making it unfeasible to guess. The name is opaque to Consul and human readable. The type is either "client" (meaning the token cannot modify ACL rules) or "management" (meaning the token is allowed to perform all actions).  

Enforcement is always done by the server nodes. All servers must be configured to provide an `acl_datacenter` which enables ACL enforcement but also specifies the authoritative datacenter. Consul does not replicate data cross-WAN and instead relies on RPC forwarding to support Multi-Datacenter configurations. However, because requests can be made across datacenter boundaries, ACL tokens must be valid globally. To avoid replication issues, a single datacenter is considered authoritative and stores all the tokens.  

When a request is made to a server in a non-authoritative datacenter server, it must be resolved into the appropriate policy. This is done by reading the token from the authoritative server and caching the result for a configurable `acl_ttl`. The implication of caching is that the cache TTL is an upper bound on the staleness of policy that is enforced. It is possible to set a zero TTL, but this has adverse performance impacts, as every request requires refreshing the policy via a cross-datacenter WAN call.  

The Consul ACL system is designed with flexible rules to accommodate for an outage of the `acl_datacenter` or networking issues preventing access to it. In this case, it may be impossible for servers in non-authoritative datacenters to resolve tokens. Consul provides a number of configurable `acl_down_policy` choices to tune behavior. It is possible to deny or permit all actions or to ignore cache TTLs and enter a fail-safe mode. The default is to ignore cache TTLs for any previously resolved tokens and to deny any uncached tokens.  

ACLs can also act in either a whitelist or blacklist mode depending on the configuration of `acl_default_policy`. If the default policy is to deny all actions, then token rules can be set to whitelist specific actions. In the inverse, the allow all default behavior is a blacklist where rules are used to prohibit actions. **By default, Consul will allow all actions.**


### Gossip encryption
The key can be set via the encrypt parameter: the value of this setting is a configuration file containing the encryption key.

The key must be 16-bytes, Base64 encoded. As a convenience, Consul provides the `consul keygen` commmand to generate a key.  
The key **must** be used on both servers and clients; this will prevent rogue consul client to join you server cluster.  

The provided key is automatically persisted to the data directory and loaded automatically whenever the agent is restarted. This means that to encrypt Consul's gossip protocol, this option only needs to be provided once on each agent's initial startup sequence. If it is provided after Consul has been initialized with an encryption key, then the provided key is ignored and a warning will be displayed. To enable encryption then, the `serf` folder need to be deleted.

The key is stored in `"${CONSUL_DATA_DIR}"/serf/local.keyring`.

### TLS encryption
```bash
openssl req -x509 -newkey rsa:2048 -days 3650 -nodes -out ca.cert

[ ca ]
default_ca = myca

[ myca ]
unique_subject = no
new_certs_dir = .
certificate = ca.cert
database = certindex
private_key = privkey.pem
serial = serial
default_days = 3650
default_md = sha1
policy = myca_policy
x509_extensions = myca_extensions

[ myca_policy ]
commonName = supplied
stateOrProvinceName = supplied
countryName = supplied
emailAddress = optional
organizationName = supplied
organizationalUnitName = optional

[ myca_extensions ]
basicConstraints = CA:false
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always
keyUsage = digitalSignature,keyEncipherment
extendedKeyUsage = serverAuth,clientAuth
```

# Troubleshooting
**Join a cluster with the wrong encryption key**
```bash
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Joining cluster...
==> EOF
```

# Sources
- [https://www.consul.io](https://www.consul.io)
- [https://www.mauras.ch/securing-consul.html](https://www.mauras.ch/securing-consul.html)
- [https://github.com/hashicorp/consul/issues/1013](https://github.com/hashicorp/consul/issues/1013)
- [https://www.digitalocean.com/community/tutorials/how-to-secure-consul-with-tls-encryption-on-ubuntu-14-04](https://www.digitalocean.com/community/tutorials/how-to-secure-consul-with-tls-encryption-on-ubuntu-14-04)
