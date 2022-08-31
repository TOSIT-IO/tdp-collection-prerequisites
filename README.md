# TDP collection prerequisites

## Supported distribution

- Centos 7
- Rocky 8.6+
- Alma[^alma] 8

[^alma]: Alma does not provide `openldap-servers` package so it is not supported for host in `[kdc]` Ansible group.

## Topology

The `topology.ini` file includes Ansible groups to work with playbooks inside this collection which can be added to your Ansible inventory configuration.

## Usage

```bash
ansible-playbook playbooks/all.yml
```

This playbook deploys the following services: Chrony, a CA, a LDAP, a KDC, a PostgreSQL.

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
Launches a kdcproxy on the `[kdcproxy]` group hosts.

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

A krb5 configuration file, parametrized for HTTPS can be found at `/etc/krb5-https.conf` on every hosts. To authenticate through https and see logs:
```bash
env KRB5_TRACE=/dev/stdout KRB5_CONFIG=/etc/krb5-https.conf kinit <your_user>
```

## PostgreSQL

Launches a PostgreSQL server on the `[postgresql]` group hosts.

### Public repository

By default, PostgreSQL PGDG public repositories are configured, you can disable it with `postgresql_use_public_repo: no` and configure your own mirror for these repositories.

### Server configuration

You can configure listen adresses, port and password encryption with `postgresql_listen_addresses`, `postgresql_port`, and `postgresql_password_encryption`.

Other settings can be set with `postgresql_additional_settings`.

### Generated settings, users, databases, schemas and pg_hba entries

Needed settings, users, databases, schemas and pg_hba entries are automatically created by reading Ansible groups. You can disable it by setting an empty array or your custom needs with these variables:

- `postgresql_settings`
- `postgresql_users`
- `postgresql_databases`
- `postgresql_schemas`
- `postgresql_hba_entries`

If you want to add your custom needs on top of generated ones use these variables:

- `postgresql_additional_settings`
- `postgresql_additional_users`
- `postgresql_additional_databases`
- `postgresql_additional_schemas`
- `postgresql_additional_hba_entries`

See `roles/postgresql/defaults/main.yml`.

The Ansible groups read are:

- `ranger_admin`
- `ranger_kms`
- `hive_ms`

You can configure default values for user, password and database with:

- Ranger Admin
    - `postgresql_rangeradmin_user`
    - `postgresql_rangeradmin_password`
    - `postgresql_rangeradmin_database`
- Ranger KMS
    - `postgresql_rangerkms_user`
    - `postgresql_rangerkms_password`
    - `postgresql_rangerkms_database`
- Hive
    - `postgresql_hive_user`
    - `postgresql_hive_password`
    - `postgresql_hive_database`

For pg_hba entries, the address can be specified and default to `all`, Ansible will read the `hostvars` `ip` to get it. You can configure the `hostvars` key read with `postgresql_generated_hba_address_hostvars_key`.

With inventory example below, 2 pg_hba entries will be generated and fill with `ip` value.

```ini
master-02 ip=192.168.56.12
master-03 ip=192.168.56.13

[hive_ms]
master-02
master-03
```

`pg_hba.conf` generated:

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    hive            hive            192.168.56.12/32        md5
host    hive            hive            192.168.56.13/32        md5
```

If you have a custom variable for the address, you can set it with `postgresql_generated_hba_address_hostvars_key: ip_data` with inventory example below.

```ini
master-02 ip=192.168.56.12 ip_data=172.16.56.12
master-03 ip=192.168.56.13 ip_data=172.16.56.13

[hive_ms]
master-02
master-03
```

`pg_hba.conf` generated:

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    hive            hive            172.16.56.12/32         md5
host    hive            hive            172.16.56.13/32         md5
```
