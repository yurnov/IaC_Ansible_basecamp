# Ansible Vault

Ansible Vault encrypts variables and files so you can protect sensitive content such as passwords or keys rather than leaving it visible as plaintext in playbooks or roles. To use Ansible Vault you need one or more passwords to encrypt and decrypt content. If you store your vault passwords in a third-party tool such as a secret manager, you need a script to access them. Use the passwords with the ansible-vault command-line tool to create and view encrypted variables, create encrypted files, encrypt existing files, or edit, re-key, or decrypt files. You can then place encrypted content under source control and share it more safely.

## Creating encrypted variables

Here is examlpe of creating encrypted variables:

```
jin@tataouine:~$ cat test.yaml
foo: bar
jin@tataouine:~$ ansible-vault encrypt --ask-vault-pass test.yaml
New Vault password:
Confirm New Vault password:
Encryption successful
jin@tataouine:~$ cat test.yaml
$ANSIBLE_VAULT;1.1;AES256
37356439616135373861663633336161356436393131313933663430353632663561303138383062
3335353738623539633637393030323464333934643835640a313462363733663031326435303164
38313438313234323366643635336161393336323234653164646561646366306163336566353034
6131366539316632610a663635316563396337363864623663613165356563633937663639353535
3832
jin@tataouine:~$ ansible-vault decrypt --ask-vault-pass test.yaml
Vault password:
Decryption successful
jin@tataouine:~$ cat test.yaml
foo: bar
jin@tataouine:~$
```

As well, vault file can be used:

```
ansible-vault encrypt --vault-password-file password.txt test.yaml
```

## Using a ansible-vault

If all the encrypted variables and files your task or playbook needs use a single password, you can use the `--ask-vault-pass` or `--vault-password-file` cli options.

To prompt for the password:

```
ansible-playbook --ask-vault-pass site.yml
```

To retrieve the password from the /path/to/my/vault-password-file file:

```
ansible-playbook --vault-password-file /path/to/my/vault-password-file site.yml
```

To get the password from the vault password client script my-vault-password-client.py:

```
ansible-playbook --vault-password-file my-vault-password-client.py
```

## Future reading

- [ansible-vault](https://docs.ansible.com/ansible/latest/cli/ansible-vault.html)
- [Encrypting content with Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html)