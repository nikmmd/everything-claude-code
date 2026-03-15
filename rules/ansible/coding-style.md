# Ansible Coding Style

## Numbered Role/Playbook Organization

Use numbered prefixes matching the infrastructure layer convention:

```
ansible/
├── inventories/
│   ├── dev/
│   │   ├── hosts.yml
│   │   └── group_vars/
│   └── prod/
│       ├── hosts.yml
│       └── group_vars/
│
├── playbooks/
│   ├── 100-networking.yml        # Foundation
│   ├── 101-dns.yml
│   ├── 200-databases.yml         # Persistent layer
│   ├── 201-storage.yml
│   ├── 300-app-deploy.yml        # Application layer
│   ├── 301-load-balancers.yml
│   ├── 400-k8s-cluster.yml       # Clustering
│   └── 401-k8s-addons.yml
│
├── roles/
│   ├── common/                   # Base role (all hosts)
│   ├── hardening/
│   ├── docker/
│   ├── nginx/
│   └── monitoring/
│
└── ansible.cfg
```

**Layer numbering** (same as Terraform):
| Range | Layer | Purpose |
|-------|-------|---------|
| 100-199 | Foundation | Networking, DNS, certificates, base packages |
| 200-299 | Persistent | Database setup, storage mounts, backups |
| 300-399 | Application | App deployment, reverse proxies, services |
| 400-499 | Clustering | K8s bootstrap, node config, addons |

## Naming

- Roles: `snake_case` — `install_docker`, `configure_nginx`
- Variables: `snake_case` with role prefix — `nginx_worker_processes`
- Tasks: Start with verb — `Install packages`, `Configure firewall`
- Files: `kebab-case` or `snake_case` consistently

## YAML Style

```yaml
# GOOD: Explicit, readable
- name: Install required packages
  ansible.builtin.apt:
    name: "{{ item }}"
    state: present
    update_cache: true
  loop: "{{ required_packages }}"
  become: true

# BAD: Shorthand, hard to read
- apt: name={{ item }} state=present
  with_items: "{{ packages }}"
```

## Best Practices

- Always use FQCNs (`ansible.builtin.apt`, not `apt`)
- Use `become: true` explicitly, never run entire playbooks as root
- Prefer `ansible.builtin.template` over `ansible.builtin.copy` for config files
- Use `block/rescue/always` for error handling
- Tag tasks for selective execution

```yaml
- name: Deploy application
  tags: [deploy, app]
  block:
    - name: Pull latest image
      community.docker.docker_image:
        name: "{{ app_image }}"
        source: pull

    - name: Start container
      community.docker.docker_container:
        name: "{{ app_name }}"
        image: "{{ app_image }}"
        state: started
        restart_policy: unless-stopped
  rescue:
    - name: Rollback to previous version
      community.docker.docker_container:
        name: "{{ app_name }}"
        image: "{{ app_image_previous }}"
        state: started
```

## Secrets

- Use `ansible-vault` for secrets, never plaintext
- Store vault password in a file excluded from git
- Use `no_log: true` for tasks handling secrets

```yaml
- name: Set database password
  ansible.builtin.lineinfile:
    path: /etc/app/config.env
    line: "DB_PASSWORD={{ db_password }}"
  no_log: true
```

## Tools

- **Linting**: `ansible-lint`
- **Syntax check**: `ansible-playbook --syntax-check`
- **Dry run**: `ansible-playbook --check --diff`
- **Vault**: `ansible-vault encrypt/decrypt/edit`
