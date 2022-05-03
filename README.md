# helm-taller
Introducción a Helm

Links de interés

[Documentación de Helm](https://helm.sh/docs/)

[Funciones internas de Helm](https://helm.sh/docs/chart_template_guide/function_list)

## Secrets Management

[Sops](https://github.com/mozilla/sops)
[Git-crypt](https://github.com/AGWA/git-crypt)

## git-crypt
This repo contains sensitive secrets that are encrypted using [git-crypt](https://www.agwa.name/projects/git-crypt/).

Encrypting files is done using git filters via a `.gitattributes` file in the directory containing
the sensitive files.
```
*.secrets.tf filter=git-crypt diff=git-crypt
```
If a file matching the `.gitattributes` glob is added to git afterwards, it will be encrypted.

### repo-setup (one time)
```bash
git-crypt init
```

### Add a new user via local keychain
If your local GPG keychain already trusts a user, we can add a user using
an existing uid.
```bash
# See all keys in the local keychain
gpg --list-keys

# Add a user by uid
git-crypt add-gpg-user --trusted victor@cloudkite.io
```

### Add a new engineer
The user needs to already have or [create a new GPG key](https://help.github.com/articles/generating-a-new-gpg-key/).

Cloudkite recommends using [Keybase](https://keybase.io/) for ease of use.

```bash
# Admin imports new user's GPG key
curl https://keybase.io/<KEYBASE ID>/pgp_keys.asc | gpg --import
# Or, if passed via file
gpg --import /path/to/gpg/file

# Add key to git-crypt
git-crypt add-gpg-user --trusted user@domain.com

# Push git-crypt to master
git push origin
```

### Removing a user
```bash
# See GPG file for all users
cd root/of/infrastructure/repo/
cd .git-crypt/keys/default/0
for file in *.gpg; do echo \"${file} : \" && git log -- ${file} | sed -n 9p; done;

git rm <GPG ID>.gpg
git commit -m 'Removing user <user>'
git push origin
```

### As a user, unlocking a repo

Once your key has been added to the repo, clone it locally and unlock it using
the following command:

```bash
git clone [...]
cd <new repo>
git-crypt unlock
```

It will prompt for your passphrase and once unlocked you can modify and commit
secrets without needing to worry about manually encrypting and decrypting.
