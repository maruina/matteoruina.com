+++
date = "2016-04-21T09:20:18+01:00"
draft = true
title = "Consul handbook"
tags = ["consul", "hashicorp"]

+++

## ACL
The ACL is Capability-based, relying on tokens to which fine grained rules can be applied. It is very similar to AWS IAM in many ways.  

## Security
### Gossip encryption
The key can be set via the encrypt parameter: the value of this setting is a configuration file containing the encryption key.

The key must be 16-bytes, Base64 encoded. As a convenience, Consul provides the `consul keygen` commmand to generate a key.  
The key **must** be used on both servers and clients; this will prevent rogue consul client to join you server cluster.  

The provided key is automatically persisted to the data directory and loaded automatically whenever the agent is restarted. This means that to encrypt Consul's gossip protocol, this option only needs to be provided once on each agent's initial startup sequence. If it is provided after Consul has been initialized with an encryption key, then the provided key is ignored and a warning will be displayed. To enable encryption then, the `serf` folder need to be deleted.

The key is stored in `"${CONSUL_DATA_DIR}"/serf/local.keyring`.

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
