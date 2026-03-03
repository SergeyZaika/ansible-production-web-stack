# WordPress Automation with Ansible (Nginx + MariaDB + PHP)

## Architecture Overview

This project provisions:

- Nginx
- MariaDB
- PHP-FPM
- Let's Encrypt (Certbot)
- Multi-site WordPress support
- Backup and restore automation

## Idempotency & Design

This project follows idempotent Ansible practices.
- No hardcoded secrets in repository
- Secrets are passed via `-e "secret_file=..."`
- Playbooks are safe to re-run
- Environment-specific data is externalized

This repository contains Ansible playbooks and templates to set up one or more WordPress sites on an Ubuntu server using Nginx, MariaDB, and PHP. Additionally, it includes a playbook for cleaning up the environment if you need to reinstall, as well as a backup playbook.

## Requirements

- Ansible installed on your control machine
- Ubuntu server(s) with SSH access
- Open ports: 80 (HTTP), 443 (HTTPS) for web traffic
- Basic knowledge of Ansible and server configuration

## Variables

### General Variables (`vars.yaml`)

- `ubuntu_user`: The username for your Ubuntu server (default: ubuntu).
- `php_version`: Version of PHP to be installed (e.g., "8.2").
- `backup_dir`: Directory to store backup files on the remote host.

### Secret Variables

All sensitive data is stored in a single secrets file passed at runtime via `-e "secret_file=..."`.

Create your own file based on `vars_secret_example.yaml`:

```yaml
domain: "example.com"
db_name: "example_db"
wordpress_install_dir: "/var/www/html/example"

db_user: "example_user"
db_password: "change_me"
root_db_password: "change_me"

wordpress_admin_email: "admin@example.com"

wordpress_files_backup: "example-files-backup.tar.gz"
wordpress_db_backup: "example-db-backup.sql"
```

## Usage

### Install the First WordPress Site

```bash
ansible-playbook -i hosts install_wordpress.yaml \
  -e "secret_file=vars_secret.yaml" \
  --vault-password-file ~/.vault_pass.txt
```

This playbook sets up the first WordPress site at the specified domain. It installs Nginx, MariaDB, PHP, and configures the server.

### Obtain SSL Certificates

```bash
ansible-playbook -i hosts certbot.yaml \
  -e "secret_file=vars_secret.yaml" \
  --vault-password-file ~/.vault_pass.txt
```

This playbook uses Certbot to obtain and install an SSL certificate for your WordPress site.

### Install the Second WordPress Site

```bash
ansible-playbook -i hosts install_second_wordpress.yaml \
  -e "secret_file=vars_secret.yaml" \
  --vault-password-file ~/.vault_pass.txt
```

This playbook sets up a second WordPress site with its own database and user credentials, without interfering with the first site.

### Create a Backup of the WordPress Site

```bash
ansible-playbook -i hosts backup_wordpress.yaml \
  -e "secret_file=vars_secret.yaml" \
  --vault-password-file ~/.vault_pass.txt
```

This playbook will backup both the WordPress files and database for the site specified in the secrets file and store the backups on the Ansible host.

### Restore WordPress from Backup

```bash
ansible-playbook -i hosts deploy_wordpress_from_backup.yaml \
  -e "secret_file=vars_secret.yaml" \
  --vault-password-file ~/.vault_pass.txt

ansible-playbook -i hosts certbot.yaml \
  -e "secret_file=vars_secret.yaml" \
  --vault-password-file ~/.vault_pass.txt
```

### Restore Second WordPress Site from Backup

```bash
ansible-playbook -i hosts deploy_second_wordpress_from_backup.yaml \
  -e "secret_file=vars_secret.yaml" \
  --vault-password-file ~/.vault_pass.txt
```

### Clean Up the Host for Reinstallation

```bash
ansible-playbook -i hosts cleanup_host.yaml \
  -e "secret_file=vars_secret.yaml" \
  --vault-password-file ~/.vault_pass.txt
```

This playbook stops and removes all installed services (Nginx, MariaDB, PHP), and deletes configuration and data files, returning the server to a clean state.

## Templates Explanation

- `fastcgi-php.conf.j2`: Configuration snippet for handling PHP requests in Nginx.
- `fastcgi_params.j2`: Defines parameters for FastCGI processes.
- `mime.types.j2`: Provides MIME type mappings for Nginx.
- `nginx.conf.j2`: Basic configuration file for Nginx.
- `nginx_wordpress.j2`: Nginx configuration for serving the first WordPress site (HTTP only).
- `nginx_wordpress_ssl.j2`: Nginx configuration for serving the first WordPress site with SSL.
- `nginx_wordpress_ssl_second.j2`: Nginx configuration for serving the second WordPress site with SSL.
- `wp-config.php.j2`: The main configuration file for WordPress, customized for the first site.
- `wp-config_second.j2`: Configuration file for the second WordPress site.

## Example Workflow

Install the first WordPress site:

```bash
ansible-playbook -i hosts install_wordpress.yaml \
  -e "secret_file=vars_secret.yaml" \
  --vault-password-file ~/.vault_pass.txt
```

Obtain an SSL certificate:

```bash
ansible-playbook -i hosts certbot.yaml \
  -e "secret_file=vars_secret.yaml" \
  --vault-password-file ~/.vault_pass.txt
```

Install the second WordPress site:

```bash
ansible-playbook -i hosts install_second_wordpress.yaml \
  -e "secret_file=vars_secret.yaml" \
  --vault-password-file ~/.vault_pass.txt
```

Create a backup:

```bash
ansible-playbook -i hosts backup_wordpress.yaml \
  -e "secret_file=vars_secret.yaml" \
  --vault-password-file ~/.vault_pass.txt
```

Restore from backup:

```bash
ansible-playbook -i hosts deploy_wordpress_from_backup.yaml \
  -e "secret_file=vars_secret.yaml" \
  --vault-password-file ~/.vault_pass.txt

ansible-playbook -i hosts certbot.yaml \
  -e "secret_file=vars_secret.yaml" \
  --vault-password-file ~/.vault_pass.txt
```

Clean up the host:

```bash
ansible-playbook -i hosts cleanup_host.yaml \
  -e "secret_file=vars_secret.yaml" \
  --vault-password-file ~/.vault_pass.txt
```

## Secrets

This repository does not contain real secrets.

Create your own secret variables file based on:

`vars_secret_example.yaml`

Run playbooks like this:

```bash
ansible-playbook install_wordpress.yaml \
  -e "secret_file=vars_secret.yaml" \
  --vault-password-file ~/.vault_pass.txt
```
