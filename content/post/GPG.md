+++
date = "2016-04-13T09:20:18+01:00"
draft = true
title = "GPG"

+++

### Backup
```bash
gpg -a --export "${KEY_ID}" > public.key
gpg -a --export-secret-keys "${KEY_ID}" > private.key
gpg --export-ownertrust > ownertrust.txt
```

### Import private key
```bash
gpg --allow-secret-key-import --import private.key
```
