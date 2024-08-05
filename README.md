## Request a certificate with the ansible_acme role

### Install prerequisites
```
ansible-galaxy role install git+https://github.com/Sapphire-Health/ansible_acme.git
ansible-galaxy collection install community.crypto
ansible-galaxy collection install community.general

pip3 install dnspython
```

### Generate an EC key pair to use for an ACME Account
```
openssl ecparam -name prime256v1 -param_enc named_curve -genkey -noout | openssl ec -aes256
```

### Generate a private key and CSR to request a certificate
```
openssl req -new -newkey rsa:2048 -nodes -sha256 -subj "/C=US/ST=/L=/O=Example Company/OU=/CN=/CN=example.com" -config <(
cat <<-EOF
[req]
default_bits = 2048
default_md = sha256
req_extensions = req_ext
distinguished_name = dn
[ dn ]
[ req_ext ]
subjectAltName = @alt_names
[alt_names]
DNS.1 = example.com
DNS.2 = www.example.com
EOF
)
```

### Create a playbook

```
cat << EOF > main.yml
---
- name: Ansible ACME
  hosts: localhost
  tasks:
    - name: Run playbook roles
      include_role:
        name: ansible_acme
EOF
```

### Create a variable file

```
cat << EOF > vars.yml
acme_key: |
    -----BEGIN EC PRIVATE KEY-----
    Proc-Type: 4,ENCRYPTED
    DEK-Info: AES-256-CBC,9E2170C94155F1BDB783C4FF8BA0CF3D

    USE PRIVATE KEY GENERATED FROM OPENSSL COMMAND ABOVE Hp3Y92Tuqn/
    /HvSdgqJkSlVkJG42QlwmcLVkdeUHxNHtOGj0O7SaoNL7e4KBIAi6FTWCCDBQPn0
    4o+OohdPEDfm11w7M7qZFiaLxNeyqkXHq6RyxuqRyZg=
    -----END EC PRIVATE KEY-----
acme_key_passphrase: "PASSWORD FROM OPENSSL COMMAND ABOVE"
acme_account_email: acme.account@fqdn.com
acme_directory: https://acme-staging-v02.api.letsencrypt.org/directory
acme_version: 2
email_recipient: email.recipient@fqdn.com
godaddy_api_key: yourgodaddyapikey
godaddy_api_secret: yourgodaddyapisecret
dns_provider: godaddy
dns_wait_timeout: 300
verify_dns: true
print_certificate: true
csr: |
    -----BEGIN CERTIFICATE REQUEST-----
    USE CSR FROM OPENSSL COMMAND OUTPUT ABOVE NVBAoMD0V4YW1wbGUgQ29t
    cGFueTEUMBIGA1UEAwwLZXhhbXBsZS5jb20wggEiMA0GCSqGSIb3DQEBAQUAA4IB
    DwAwggEKAoIBAQCJnakjYL8aD8eWaueU+YsLiz97peRXn6OymKlBtzE3K520RO2U
    eCgIpwGOX/E0lx3nswyhRDpqgzjCS864E23lXR/Y3812MYRLU65rt/vRIIc7lntO
    V8c8pgV4Xq67Phaw558DC8LJtX5M0sExiSzEEqMr3/plXhMf1ShDji9Ctcry/M03
    rhta9kFuZKzYholL6hdb6fGKqOGU7k1RhkGEn7cB16fZoVFLgi+ZOVKD6s/eMAhc
    wUcxe+F3TnZakHR5ijrRBLlePc2hP/eIQzbcDo8rWvHfJ+K9wWOYPlo7nxprPWQk
    vnscEQhfcq4kRc0E1NX6a30P0VwCHF0tgQo1AgMBAAGgOjA4BgkqhkiG9w0BCQ4x
    KzApMCcGA1UdEQQgMB6CC2V4YW1wbGUuY29tgg93d3cuZXhhbXBsZS5jb20wDQYJ
    KoZIhvcNAQELBQADggEBAHNcD/XlpTmdHGUPqrKKB8zSo3U2vMKqCf0rXsaB+f2Y
    OJeKNNIu5dZn4xZ/RRWwy2vOBzxLB97caL8OrQ1yNJQSj7LXwFa6qRFj0LrnAgXI
    UgkW3BuLZMtUYq7SjONxCzSSLpHJSn/5Wkk9xZTJgCnK0WSeZsSgL/Xj4vo8oWxr
    Xs6X1QZRGDpJoT0edNpA6h0/aOZuf00bto1vaPDq5fgmniZ9C+lmRrfNXyRlo0Ie
    obI/KeynwSNHyvE3Exy7E76OJJlE7Truyoeqbl84QE+Dn7UiKhQU/yoLhV6kyVfo
    twtTlq1DnAUmlNp91Z6myP78A2yAubeOcNLx6kTylWo=
    -----END CERTIFICATE REQUEST-----
EOF
```
### Run the playbook
```
ansible-playbook -e @vars.yml main.yml
```