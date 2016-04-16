+++
date = "2016-04-13T09:20:18+01:00"
draft = false
title = "GPG"
tags = ["security"]

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

### Sign git commit
To set all commits for a repository to be signed by default, in Git versions 2.0.0 and above, run `git config commit.gpgsign true`. To set all commits in any local repository on your computer to be signed by default, run `git config --global commit.gpgsign true`.  
To sign a commit run `git commit -S -m "${COMMIT_MESSAGE}"`.
