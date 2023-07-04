# Ansible Role: lego

This role runs [acme-lego](https://go-acme.github.io/lego/) on the localhost, such that the acme account and DNS api credentials are never communicated to the server. It also creates boilerplate nginx configuration in accordance with the Mozilla's recomended TLS configuration.

This role supports using multiple providers at the same time, just source all the credentials needed beforehand.

## Requirements

The `nginx_config` role which is distributed in the nginx_core collection.

```yaml
collections:
  - name: nginxinc.nginx_core
```

It's not listed in the `meta` dependencies since that would run the role out of sequence.

## Role Variables

You can configure multiple providers and domains with a single ACME account.

```yaml
acme_email: acme@example.com
acme_domains:
  - { domain: myhost.example.com, provider: easydns }
```

Lego uses environment variables to authenticate to your DNS provider. You should source those secrets as environment variables before running the playbook.

If for some reason you cannot source the environment variables ahead of running the playbook, you can define them as Ansible variables.

```yaml
lego_environment:
  - NAMECHEAP_API_USER: '...'
  - NAMECHEAP_API_KEY: '...'
```

## Secrets

The api keys are sprinkled throughout the task as environment variables until I come up with a smarter way to do that.

## File Permissions

File access to the certificates and keys are controlled by way of unix permissions. The files are owned by the acme system user/group, and each service that needs to use the certificates just need to belong to the acme group.

The playbook could be used to renew certificates from multiple DNS providers. The only provider I'm using currently is Name cheap. You will need to edit the environment variables in the following files if you want to use other providers:

* main.yml
* tasks/certificates.yml

You can test your results: ssllabs.com/ssltest

## Example Playbook

```yaml
- hosts: all
  roles:
    - bleetube.lego
```
