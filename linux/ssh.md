# ssh config file example

```bash
Host github
    HostName github.com
    User git
    IdentityFile ~/.ssh/github
```

# copy private key in file

## Create the file, copy the key, then save and exit
```bash
nano ~/.ssh/github
```

## change rights
```bash
chmod 600 ~/.ssh/github
```