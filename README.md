ansible-role-ssl-certs
======================

Generate and/or deploy SSL certificates

Available on Ansible Galaxy: [jdauphant.ssl-certs](https://galaxy.ansible.com/list#/roles/3115)

# Examples

## Minimal example to generate a self-signed SSL certificate

```YAML
- hosts: all
  roles:
    - jdauphant.ssl-certs
```

This will create certificate and private key in:

- `/etc/ssl/myserver.mydomain.com.key`
- `/etc/ssl/myserver.mydomain.com.pem`

## Example to generate and deploy a self-signed SSL certificate

```YAML
- hosts: all
  roles:
    - role: jdauphant.ssl-certs
      ssl_certs:
        - fields: "/C=US/ST=New York/L=New York/O=Your company/CN=example.com"
          privkey_path: "{{ ssl_certs_path }}/example.com.key"
          cert_path: "{{ ssl_certs_path }}/example.com.pem"
          csr_path: "{{ ssl_certs_path }}/example.com.csr"
```

The certificate has to be placed in `files/ssl/example.com.key` and `files/ssl/example.com.pem`. If
they don't exist, the key and a **self-signed** certificate will be generated at
`/etc/ssl/example.com/example.com.key` and `/etc/ssl/example.com/example.com.pem` using the provided common name.


## Example to deploy a SSL certificate using local key/pem files

```YAML
- hosts: all
  roles:
    - role: jdauphant.ssl-certs
      ssl_certs:
        - privkey_path: "{{ ssl_certs_path }}/example.com.key"
          cert_path: "{{ ssl_certs_path }}/example.com.pem"
          local_privkey_path: '/path/to/example.com.key'
          local_cert_path: '/path/to/example.com.pem'
```

## Example to deploy a SSL certificate stored in variables

An SSL certificate and key are just text that can be stored as a variable, which is useful when
using ansible vault.

Example variable data, note how the text blob is indented. This is needed to correctly insert the
text via the template module.

```YAML
- hosts: all
  roles:
    - role: jdauphant.ssl-certs
      ssl_certs:
        - privkey_path: "{{ ssl_certs_path }}/example.com.key"
          cert_path: "{{ ssl_certs_path }}/example.com.pem"
          local_privkey_data: |
            -----BEGIN RSA PRIVATE KEY-----
            MIIEpQIBAAKCAQEAu2uhv2cjoN4F3arUZ5cDrwuxf3koCwrKSK75as0WZoxYrpyw
            Lyx9ldyD4nGabVep0R/uAgQ/HqEf2jC7WIvGcEq8bHB9PyEEWzT8IjKQX0YTc//4
            gkHBkpyU0fVrj5nkc30EIbcbH4RHRDwye4VhP/iCPchDG7OqvCyOdm8=
            -----END RSA PRIVATE KEY-----
          local_cert_data: |
            -----BEGIN CERTIFICATE-----
            MIIDmzCCAoOgAwIBAgIJAKWMlgLwrBzXMA0GCSqGSIb3DQEBCwUAMGQxCzAJBgNV
            QAL3naEfBSZBl0tBohuxn8Xd3yLPuKGUOk3pSL1IJy0Ca6p+QwjkaZUd9X3gf1V2
            SEfYSaGPvfIlSuHIshno
            -----END CERTIFICATE-----
```

Then simply include the role as in the first example.

## Example to deploy more than one SSL certificate at once

Sometimes is useful to deploy more than one certificate at once. You can use any combination
of self-signed certificate generation, local key/pem files or variable-stored certificates.

```YAML
- hosts: all
  roles:
    - role: jdauphant.ssl-certs
      ssl_certs:
        - privkey_path: "{{ ssl_certs_path }}/example1.com.key"
          cert_path: "{{ ssl_certs_path }}/example1.com.pem"
          local_privkey_path: '/path/to/example1.com.key'
          local_cert_path: '/path/to/example1.com.pem'
        - privkey_path: "{{ ssl_certs_path }}/example2.com.key"
          cert_path: "{{ ssl_certs_path }}/example2.com.pem"
          local_privkey_data: |
            -----BEGIN RSA PRIVATE KEY-----
            MIIEpQIBAAKCAQEAu2uhv2cjoN4F3arUZ5cDrwuxf3koCwrKSK75as0WZoxYrpyw
            Lyx9ldyD4nGabVep0R/uAgQ/HqEf2jC7WIvGcEq8bHB9PyEEWzT8IjKQX0YTc//4
            gkHBkpyU0fVrj5nkc30EIbcbH4RHRDwye4VhP/iCPchDG7OqvCyOdm8=
            -----END RSA PRIVATE KEY-----
          local_cert_data: |
            -----BEGIN CERTIFICATE-----
            MIIDmzCCAoOgAwIBAgIJAKWMlgLwrBzXMA0GCSqGSIb3DQEBCwUAMGQxCzAJBgNV
            QAL3naEfBSZBl0tBohuxn8Xd3yLPuKGUOk3pSL1IJy0Ca6p+QwjkaZUd9X3gf1V2
            SEfYSaGPvfIlSuHIshno
            -----END CERTIFICATE-----
```

## Example to use this role with my Nginx role: [jdauphant.nginx](https://github.com/jdauphant/ansible-role-nginx)

```YAML
- hosts: all
  roles:
    - jdauphant.ssl-certs
      ssl_certs_generate_dh_param: true
      ssl_certs:
        - fields: "/C=US/ST=New York/L=New York/O=Your company/CN=example.com"
          privkey_path: "{{ ssl_certs_path }}/example.com.key"
          cert_path: "{{ ssl_certs_path }}/example.com.pem"
          csr_path: "{{ ssl_certs_path }}/example.com.csr"
    - role: jdauphant.nginx
      nginx_configs:
        ssl:
          - ssl_certificate_key {{ ssl_certs[0].privkey_path }}
          - ssl_certificate {{ ssl_certs[0].cert_path }}
          - ssl_dhparam {{ ssl_certs_dhparam_path }}
      nginx_sites:
        default:
          - listen 443 ssl
          - server_name _
          - root "/usr/share/nginx/html"
          - index index.html
```
