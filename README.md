# TDP collection prerequisites

## Supported distribution

- Centos 7
- Rocky 8.6+, 9
- Alma[^alma] 8, 9

[^alma]: Alma does not provide `openldap-servers` package so it is not supported for host in `[kdc]` Ansible group.

## Topology

The `topology.ini` file includes Ansible groups to work with playbooks inside this collection which can be added to your Ansible inventory configuration.

## Usage

```bash
ansible-playbook playbooks/all.yml
```

This playbook deploys the following services: Chrony, a CA.

## HTTP proxy configuration

You can configure HTTP proxy with Ansible variables:
- `http_proxy`
- `https_proxy`
- `no_proxy`

## System configuration

Populate `/etc/hosts` file for `[all]` Ansible hosts. You need to specify the IP used with the `ip` Ansible variable for each host, same for the `domain`.

Configure `yum`/`dnf` global http proxy and install needed `yum`/`dnf` packages.

Configure `JAVA_HOME` variable inside `/etc/environment`.

Disable `firewalld`.

Install needed Python packages with `pip3` for PySpark.

```bash
ansible-playbook playbooks/system.yml
```

## Chrony (NTP)

Configure Chrony as NTP client.

```bash
ansible-playbook playbooks/chrony.yml
```

You can configure NTP servers with `ntp_servers` Ansible variable.

## Certificate Authority and Certificates

Creates a certificate authority at the `[ca_host]` Ansible group and distributes signed certificates and keys to each VM.

```bash
ansible-playbook playbooks/certificates.yml
```

_The certificates will also be downloaded to the `roles/certificates/files/tdp_getting_started_certs` local project folder._

## LDAP and Kerberos

Launches a LDAP server and KDC on the `[kdc]` group hosts.

On each host installs Kerberos clients and enable SSSD LDAP authentification.

Create `ldap_groups` and `ldap_users` specified. For each users, a Kerberos principal is created and a keytab is generated in `/home/<username>/<username>.keytab` on  the`[users_keytab]` group hosts.

You can configure ticket renewal lifetime:
- `kerberos_renew_lifetime`: the default renew lifetime asked for during a kinit, templated in krb5.conf
- `kerberos_max_renewable_life`: the default max renewable life parameter of any principal created, templated in kdc.conf

Note that some tools may malfunction if `kerberos_renew_lifetime` > `kerberos_max_renewable_life`.

```
ansible-playbook playbooks/ldap_kerberos.yml
```

_After this, you can log in as the Kerberos admin from any VM with the command `kinit admin/admin` and the password `admin123`. When using a user unix account on `[users_keytab]` group hosts, you can log in with `kinit -ki` which will use the keytab inside the home directory._
