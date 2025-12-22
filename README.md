# GitLab-Integration-with-LDAP-Authentication
Deploy GitLab with LDAP authentication and secure HTTPS using a self-signed certificate on Linux.
---
# 🚀 GitLab Integration with LDAP Authentication (Secure HTTPS Setup)

----------

### Objective
The objective of this configuration is to deploy **GitLab** on a Linux server and integrate it with an **LDAP server** for centralized authentication. GitLab is secured using **HTTPS** with a self-signed SSL certificate and made accessible using a custom domain name.
___


## 1. Server Preparation & GitLab Installation (Ubuntu/Debian)

```bash
apt update && apt upgrade -y
```
```
apt install -y curl ca-certificates openssh-server
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | bash
apt install gitlab-ee
```

## 2. Create SSL Directory & Generate Self-Signed Certificate

```bash
sudo mkdir -p /etc/gitlab/ssl
```
```
sudo chmod 700 /etc/gitlab/ssl
```
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:4096 \
  -keyout /etc/gitlab/ssl/git.example.local.key \
  -out /etc/gitlab/ssl/git.example.local.crt
```

When prompted, enter the following (most important is Common Name):

```
Common Name (CN): git.example.local
```

All other fields → just press Enter

Verify files created:

```bash
ls -l /etc/gitlab/ssl/
# Output should show:
# git.example.local.crt
# git.example.local.key
```

## 3. Configure GitLab (/etc/gitlab/gitlab.rb)

```bash
sudo nano /etc/gitlab/gitlab.rb
```

- Paste the following configuration **as-is** and then replace placeholders:

```ruby
# gitlab_backup_cli['dir'] = '/var/opt/gitlab/backups'
# gitlab_backup_cli['additional_groups'] = %w[git gitlab-psql registry]

# Enable LDAP authentication
gitlab_rails['ldap_enabled'] = true

# LDAP server configuration
gitlab_rails['ldap_servers'] = {
  'main' => {
    'label' => 'LDAP',
    'host' => '<LDAP_SERVER_IP>',              # Enter your LDAP server IP address
    'port' => 389,                             # Default LDAP port
    'uid'  => 'uid',

    'bind_dn' => 'cn=admin,dc=example,dc=local',
    # Enter your LDAP admin DN here

    'password' => '<LDAP_ADMIN_PASSWORD>',
    # Enter your LDAP admin password here

    'encryption' => 'plain',                   # Plain LDAP (389)
    'base' => 'dc=example,dc=local'             # Enter your LDAP base DN
  }
}

# GitLab external URL (default HTTPS)
external_url "https://git.example.local"
# Replace with your GitLab domain name

# GitLab internal NGINX HTTPS configuration (default port 443)
nginx['listen_https'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/git.example.local.crt"
# Path to SSL certificate file

nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/git.example.local.key"
# Path to SSL private key file

# Disable Let's Encrypt (private/local domain usage)
letsencrypt['enable'] = false

```

Save & exit (`Ctrl+O` → `Enter` → `Ctrl+X`)

## 4. Apply Configuration

```bash
sudo gitlab-ctl reconfigure
```
```
sudo gitlab-ctl restart
```

Wait 2–4 minutes for everything to start.

## 5. Test LDAP Connection

```bash
sudo gitlab-rake gitlab:ldap:check
```

**Expected Output:**
```
Expected Output:
LDAP authentication... Success
```

## 6. Final Access Test

From any machine on the network (including the server itself):

```bash
curl -k https://git.example.local
```
### 🌐 Access GitLab
Or open in browser:
```
https://git.example.local
```

---
> Use this section only if port 443 is already in use (for example, by Apache).

---

## Optional: Run GitLab on Custom Port (e.g., 8443)

- Paste the following configuration **as-is** and then replace placeholders:
```bash
sudo nano /etc/gitlab/gitlab.rb
```


```
# gitlab_backup_cli['dir'] = '/var/opt/gitlab/backups'
# gitlab_backup_cli['additional_groups'] = %w[git gitlab-psql registry]

# Enable LDAP authentication
gitlab_rails['ldap_enabled'] = true

# LDAP server configuration
gitlab_rails['ldap_servers'] = {
  'main' => {
    'label' => 'LDAP',
    'host' => '<LDAP_SERVER_IP>',          # Enter your LDAP server IP address
    'port' => 389,                         # Default LDAP port
    'uid'  => 'uid',
    'bind_dn' => 'cn=admin,dc=example,dc=local', 
    # Enter your LDAP admin distinguished name

    'password' => '<LDAP_ADMIN_PASSWORD>', 
    # Enter your LDAP admin password here

    'encryption' => 'plain',               # Use plain for LDAP (389)
    'base' => 'dc=example,dc=local'         # Enter your LDAP base DN
  }
}

# GitLab external URL configuration
external_url "https://git.example.local:8443"
# Replace with your GitLab domain name

# GitLab internal NGINX HTTPS configuration
nginx['listen_port'] = 8443
nginx['listen_https'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/git.example.local.crt"
# Path to SSL certificate file

nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/git.example.local.key"
# Path to SSL private key file

# Disable Let's Encrypt (used only for public domains)
letsencrypt['enable'] = false

```

Then reconfigure:

```bash
sudo gitlab-ctl reconfigure
```
```
sudo gitlab-ctl restart
```

Allow the port if firewall is active:

```bash
sudo ufw allow 8443/tcp
```
### 🌐 Access GitLab
Or open in browser:
```
https://git.example.local:8443
```

## Done!  
Your GitLab is now running with LDAP authentication and HTTPS using your own domain and IP.  
Login with any LDAP user on first access 

---
[Krishna kushwah ](https://www.linkedin.com/in/krishna-kushwah-382812317/)
