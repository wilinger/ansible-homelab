[![CI](https://github.com/wilinger/ansible-homelab/actions/workflows/ci.yml/badge.svg)](https://github.com/wilinger/ansible-homelab/actions/workflows/ci.yml)


<!-- ABOUT THE PROJECT -->
## Configuration Management with Ansible

Example Ansible playbooks and roles used to configure k8s nodes and various servers in my homelab with security in mind.

### Free and Open Source Tools Used
* Authenticating to hosts with short lived [Signed SSH Certificates](https://www.vaultproject.io/docs/secrets/ssh/signed-ssh-certificates)
* Retrieving secrets with [Vault AppRole Auth Method](https://www.vaultproject.io/docs/auth/approle) and [hashi_vault plugin](https://docs.ansible.com/ansible/latest/collections/community/hashi_vault/hashi_vault_lookup.html)
* Ansible playbook testing with [Molecule](https://molecule.readthedocs.io/en/latest/)
* Ansible linting with [yamllint](https://github.com/adrienverge/yamllint) and [ansible-lint](https://ansible-lint.readthedocs.io/en/latest/)
* Pre-commit hook to prevent commiting secrets with [git-secrets](https://github.com/awslabs/git-secrets)
* Secrets detection with [ggshield](https://github.com/GitGuardian/ggshield)
<p align="right">(<a href="#top">back to top</a>)</p>


<!-- Details -->
## Implementation Overview

### Secure authentication to hosts with short lived Signed SSH Certificates

Using personal SSH private keys to authenticate to servers raises the risk of unauthorized access due to key compromise. Using Signed SSH Certificates can mitigate this risk and provides additional security benefits over traditional SSH with personal private keys:

* Mitigate risk of unauthorized access due to private key compromise. Private keys poses a security risk and are targets for threat actors due to the possibility of users intentionally or unintentionally exposing them to other users. With Signed SSH Certificates, no personal public keys are added to a user's authorized_keys file, thus preventing access via personal user private keys.
* Signed SSH Certificates can be short lived.  Making signed SSH Certificates short lived makes it a lot safer than traditional longer term personal SSH private keys.  Even if a threat actor were to compromise the signed certificates within the alotted time the certificate is valid for, the risk of unauthorized access can be lowered with [Host Key Signing and adding valid principles.](https://www.vaultproject.io/docs/secrets/ssh/signed-ssh-certificates)
* No need for key rotation. With Signed SSH Certificates, there is no longer a need to worry about rotating keys to meet compliance requirements or when a user leaves the company.
* Better scalability. Managing personal keys across multiple systems and cloud environments consistently can be a complex operation. Using Vault as an SSH certificate authority, you can allow access via an external authentication source such as LDAP or OIDC authentication.

For my homelab, hosts are configured with [Vault SSH CA host role](roles/ssh-ca-host/tasks/main.yml) to accept connections with signed certificates.  
Authentication to hosts is achieved with [Vault SSH CA client role](roles/ssh-ca-client/tasks/main.yml).  
Refer to [Github Actions ansible workflow](.github/workflows/ansible.yml) and the actual [run](https://github.com/wilinger/ansible-homelab/runs/4820408902?check_suite_focus=true) to see it in action.  


### Retrieving secrets for roles
In addition to using Vault for Signed SSH Certificates, Vault is also used to retrieve secrets for Ansible role variables with [Vault AppRole Auth Method](https://www.vaultproject.io/docs/auth/approle) and [hashi_vault plugin](https://docs.ansible.com/ansible/latest/collections/community/hashi_vault/hashi_vault_lookup.html)
The benefit of using Hashicorp Vault over Ansible's vault:
* No need to store the vault password on a file locally or passed through the command line.
* Secure central management of secrets using [Vault's KV Secrets Engine](https://www.vaultproject.io/docs/secrets/kv) without decrypting and re-encrypting secrets in a file which raises the risk of accidentally committing secrets to a repositry.

Example usage of using a Vault AppRole to retrieve secrets can be seen in the [k8s.yml](k8s.yml) playbook using the [hashi_vault plugin](https://docs.ansible.com/ansible/latest/collections/community/hashi_vault/hashi_vault_lookup.html). Note that the VAULT_ADDR, ANSIBLE_HASHI_VAULT_ROLE_ID, ANSIBLE_HASHI_VAULT_SECRET_ID environment variables are set in in a [Github Actions ansible workflow](.github/workflows/ansible.yml).  

### Molecule linting and testing
Playbooks are tested and verified using [Molecule](https://molecule.readthedocs.io/en/latest/).  This uses a custom [docker image](https://github.com/wilinger/docker-ubuntu2004-molecule) built with openssh-server installed and systemd script to execute the systemd command without systemd.

The molecule run also performs linting tests using [yamllint](https://github.com/adrienverge/yamllint) and [ansible-lint](https://ansible-lint.readthedocs.io/en/latest/).

### To do 
* CIS controls
* secrets prevention and detection write up


