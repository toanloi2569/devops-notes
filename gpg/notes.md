# Import secret gpg key (copied from one machine to another)?

**Export keys**
```
gpg --export ${ID} > public.key
gpg --export-secret-key ${ID} > private.key
```

**Import keys**
```
gpg --import public.key
gpg --import private.key
```

**List keys**
```
gpg --list-keys
```


**Encrypt file**
```
gpg -c <filepath>
```


**Decrypt file**  
```
gpg -d <filepath> > <decrypted-filepath>
```