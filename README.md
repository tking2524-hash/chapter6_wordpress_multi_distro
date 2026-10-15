# Chapter 6 Multi-Distribution WordPress Ansible Lab

This project deploys the same WordPress site to one Debian-based system and one Red Hat-based system with a single Ansible playbook. It continues to use the three Chapter 5 roles: `stack_install`, `stack_config`, and `wordpress`.

## Supported Systems

The inventory currently contains:

- Debian-based host: `wordpress_debian` at `10.2.37.5`
- Red Hat-based host: `wordpress_redhat` at `10.2.37.97`

The playbook supports hosts where `ansible_os_family` is either `Debian` or `RedHat`.

## How the Playbook Identifies the Operating System

`site.yml` uses `gather_facts: true`. The roles use the gathered `ansible_os_family` fact to apply distribution-specific values and tasks only where required.

Examples:

```yaml
when: ansible_os_family == 'Debian'
```

```yaml
when: ansible_os_family == 'RedHat'
```

The playbook also displays each host's detected distribution, version, and OS family before the roles run.

## Project Structure

```text
chapter6_wordpress_multi_distro/
├── .gitignore
├── ansible.cfg
├── hosts
├── requirements.yml
├── site.yml
├── README.md
├── troubleshooting-summary.md
├── reflection.md
├── group_vars/
│   └── wordpress_servers.yml
├── output/
├── screenshots/
└── roles/
    ├── stack_install/
    │   ├── tasks/main.yml
    │   └── handlers/main.yml
    ├── stack_config/
    │   ├── tasks/main.yml
    │   ├── handlers/main.yml
    │   └── templates/wordpress-nginx.conf.j2
    └── wordpress/
        ├── tasks/main.yml
        └── handlers/main.yml
```

## Common and Distribution-Specific Settings

Shared settings are stored in `group_vars/wordpress_servers.yml`. These include the site title, WordPress administrator settings, database settings, WordPress path, plugin list, WP-CLI path, and TLS paths.

Distribution-specific values are selected in `stack_install` after Ansible gathers facts. These include:

| Setting | Debian-based | Red Hat-based |
|---|---|---|
| Web account | `www-data` | `nginx` |
| PHP MySQL package | `php-mysql` | `php-mysqlnd` |
| Python MySQL package | `python3-pymysql` | `python3-PyMySQL` |
| MariaDB socket | `/run/mysqld/mysqld.sock` | `/var/lib/mysql/mysql.sock` |
| PHP-FPM service | versioned, e.g. `php8.x-fpm` | `php-fpm` |
| PHP-FPM socket | versioned under `/run/php/` | `/run/php-fpm/www.sock` |
| Firewall configuration | no firewall changes made by this lab | firewalld HTTP/HTTPS rules |
| SELinux configuration | not used | SELinux context and boolean tasks |

The NGINX template uses the distribution-specific `php_fpm_socket` variable so the same template works on both hosts.

## Required Collections

Install the collections before running the playbook:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Inventory Test

Confirm Ansible can reach both hosts:

```bash
ansible wordpress_servers -m ping
```

Both hosts should return `SUCCESS` and `pong`.

## Syntax Check

```bash
ansible-playbook site.yml --syntax-check
```

## Deployment

Run the deployment against both systems and save the output:

```bash
ansible-playbook site.yml --ask-become-pass | tee output/deployment-success.txt
```

The final recap should show both hosts with:

- `unreachable=0`
- `failed=0`

Do not copy example recap numbers into the submission. Use the actual output produced by your systems.

## WordPress Verification

Open both sites in a browser:

- `https://10.2.37.5`
- `https://10.2.37.97`

The certificates are self-signed, so a browser security warning is expected in this lab.

Capture one screenshot of each WordPress site and save them in `screenshots/`.

Verify the title on both hosts:

```bash
ansible wordpress_servers -b --ask-become-pass -m shell -a "wp option get blogname --path=/var/www/wordpress --allow-root"
```

Verify plugins:

```bash
ansible wordpress_servers -b --ask-become-pass -m shell -a "wp plugin list --path=/var/www/wordpress --allow-root"
```

The required plugins are:

- Akismet
- Yoast SEO (`wordpress-seo`)

## Idempotence Test

Immediately run the playbook a second time:

```bash
ansible-playbook site.yml --ask-become-pass | tee output/idempotence-test.txt
```

Review the recap and any task that still reports `changed`. The goal is for the second run to have no unnecessary changes. Record the actual recap from your environment in the submission.

## Controlled-Failure Test

The required controlled failure is documented in `troubleshooting-summary.md`.

A simple test is to temporarily replace the Red Hat package `php-mysqlnd` with the Debian-style package name `php-mysql`, run the playbook, save the error, restore `php-mysqlnd`, and run it again successfully.

This demonstrates why distribution-specific package variables are necessary.

## Important Platform Differences

The largest differences are PHP-FPM naming and paths, package names, database socket paths, and Red Hat security configuration. Debian commonly uses a versioned PHP-FPM service and socket, whereas Red Hat uses `php-fpm` and `/run/php-fpm/www.sock`. Red Hat also requires the firewalld and SELinux tasks included in this project.

## Known Limitations

- The project is currently designed only for Debian-family and Red Hat-family systems.
- TLS uses a self-signed certificate intended for the lab.
- The WordPress and database passwords in `group_vars/wordpress_servers.yml` are placeholders and must be changed before running the lab.
- A production deployment should store secrets with Ansible Vault.
- The Debian host is not automatically configured with UFW rules. If UFW is already enabled, HTTP/HTTPS access must be permitted separately.
- The playbook assumes MariaDB uses the normal distribution socket locations configured in the role.

## Troubleshooting Guidance

### Ansible cannot connect

Run:

```bash
ansible wordpress_servers -m ping
```

Verify the IP address, SSH username, network connectivity, and SSH authentication.

### PHP-FPM service failure

Display the detected platform and PHP service information by running the playbook with verbose output:

```bash
ansible-playbook site.yml --ask-become-pass -vvv
```

On Debian, confirm the installed PHP version matches the versioned PHP-FPM service under `/etc/php/` and `/run/php/`.

### NGINX configuration failure

Run:

```bash
ansible wordpress_servers -b --ask-become-pass -m command -a "nginx -t"
```

Check that the PHP-FPM socket in the generated NGINX configuration exists on each host.

### Database failure

Check MariaDB:

```bash
ansible wordpress_servers -b --ask-become-pass -m shell -a "systemctl is-active mariadb"
```

Check the socket paths if a MySQL module reports that it cannot connect locally.

### Site does not load externally

Confirm NGINX is listening:

```bash
ansible wordpress_servers -b --ask-become-pass -m shell -a "ss -ltnp | grep -E ':80 |:443 ' || true"
```

On the Red Hat host, also confirm firewalld allows HTTP and HTTPS. On Debian, check UFW manually if it is enabled.

## Suggested Git Workflow

Before making Chapter 6 changes, commit the copied Chapter 5 project:

```bash
git add .
git commit -m "Copy Chapter 5 WordPress project for Chapter 6"
```

Then make meaningful commits such as:

```bash
git add .
git commit -m "Add Debian and Red Hat hosts to inventory"
```

```bash
git add .
git commit -m "Add multi-distribution package and PHP-FPM support"
```

```bash
git add .
git commit -m "Add cross-platform WordPress validation and troubleshooting docs"
```

```bash
git add .
git commit -m "Complete Chapter 6 multi-distribution WordPress lab"
```

Push the final project to the Git repository required by the course.

## Submission Checklist

- [ ] Compressed copy of complete Ansible project
- [ ] Git repository URL
- [ ] Completed `README.md`
- [ ] Successful deployment output for both hosts
- [ ] Idempotence-test output
- [ ] Debian WordPress screenshot
- [ ] Red Hat WordPress screenshot
- [ ] Controlled-failure troubleshooting summary
- [ ] Short reflection
