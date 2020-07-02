## ansible-restund

## Requirements

- ansible >= 2.7

## Preparation

1. Download the role and its dependencies

```
curl -Ls https://raw.githubusercontent.com/wireapp/ansible-restund/master/requirements.yml > ansible-restund-requirements.yml
ansible-galaxy install -f -r ansible-restund-requirements.yml
```

2. Generate a secret as described in [how to install wire-server](https://docs.wire.com/how-to/install/helm.html#how-to-install-wire-server-itself), and store it under `restund-secret.txt`, e.g:

```
openssl rand -base64 64 | env LC_CTYPE=C tr -dc a-zA-Z0-9 | head -c 42 > restund-secret.txt
```

3. create a playbook file (see below), and run ansible-playbook against your server.

## Example playbooks

### Basic usage

Install restund to listen on default port 3478 for UDP and TCP

```ini
# hosts.ini

[all]
restund01  ansible_host=X.X.X.X

[restund]
restund01
```

```yaml
- name: Install restund
  hosts: restund
  gather_facts: yes
  become: yes
  vars:
    restund_zrest_secret: "{{ lookup('file', 'restund-secret.txt') }}"
  roles:
    - role: restund
```

### With auto-renewing TLS certificates

Also allow TLS connections on default port 5349, by configuring auto-renewing let's encrypt certificates.

* You need to have a DNS record pointing to your machine, i.e. `dig +short restund01.dev.example.com` should return the IP address of your machine.

```ini
# hosts.ini

[all]
restund01  ansible_host=X.X.X.X  certbot_domain=restund01.dev.example.com

[restund]
restund01
```

```yaml
- name: Install restund
  hosts: restund
  gather_facts: yes
  become: yes
  vars:
    restund_zrest_secret: "{{ lookup('file', 'restund-secret.txt') }}"
    certbot_enabled: true
    certbot_admin_email: # e.g. certificates@example.com
  roles:
    - role: restund
```

### With custom TLS certificates

* You need to have a DNS record pointing to your machine, i.e. `dig +short restund01.dev.example.com` should return the IP address of your machine.
* You need to have a valid certificate and private key for that domain.

1. Create a PEM file containing certificate, intermediate certificate, and private key, like this:

```
# -----BEGIN CERTIFICATE-----
# --- ... CERT CONTENT ... --
# -----END CERTIFICATE-----
# -----BEGIN CERTIFICATE-----
# --- ... INTERMEDIATE ..----
# -----END CERTIFICATE----
# -----BEGIN PRIVATE KEY-----
# --- .... PRIV KEY -----
# -----END PRIVATE KEY-----
```

save that file as `tls_cert_and_priv_key.pem`

```yaml
- name: Install restund
  hosts: restund
  gather_facts: yes
  become: yes
  vars:
    restund_zrest_secret: "{{ lookup('file', 'restund-secret.txt') }}"
    restund_tls_certificate: "{{ lookup('file', 'tls_cert_and_priv_key.pem') }}"
  roles:
    - role: restund
```

### Other overrides

See defaults/main.yml for other variables to change.

## How do I connect this restund server with wire-server?

You need to make sure to use the same secret for the ansible-restund role as you configure under `brig.secrets.turn.secret` for the wire-server helm chart. (see also [documentation](https://docs.wire.com/how-to/install/helm.html#how-to-install-wire-server-itself))

Once you have a provisioned server, take note of the advertised IP address and ports (for UDP and TCP) and then add them in your wire-server installation. I.e., if your server is now running at `a.b.c.d` and the used udp/tcp port is 3478, then add that config as examplified under `brig.turnStatic` [here](https://github.com/wireapp/wire-server-deploy/blob/master/values/wire-server/prod-values.example.yaml#L76).

**Status: beta**, see [TODO](TODO.md)

[![Build Status](https://travis-ci.org/wireapp/ansible-restund.svg?branch=master)](https://travis-ci.org/wireapp/ansible-restund)
