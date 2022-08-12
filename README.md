# TDP collection prerequisites

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
