# Ansible Patterns

## Role Structure

```
roles/my_role/
├── defaults/main.yml      # Default variables (lowest precedence)
├── vars/main.yml           # Role variables (higher precedence)
├── tasks/main.yml          # Task entry point
├── handlers/main.yml       # Handlers (service restarts, etc.)
├── templates/              # Jinja2 templates
├── files/                  # Static files
├── meta/main.yml           # Role metadata and dependencies
└── molecule/               # Testing
    └── default/
        ├── molecule.yml
        └── verify.yml
```

## Idempotent Tasks

```yaml
# GOOD: Idempotent — safe to run repeatedly
- name: Ensure nginx is installed
  ansible.builtin.apt:
    name: nginx
    state: present

- name: Ensure config is in place
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart nginx

# BAD: Not idempotent
- name: Add line to config
  ansible.builtin.shell: echo "worker_processes 4;" >> /etc/nginx/nginx.conf
```

## Handler Pattern

```yaml
# tasks/main.yml
- name: Update nginx config
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify:
    - Validate nginx config
    - Restart nginx

# handlers/main.yml
- name: Validate nginx config
  ansible.builtin.command: nginx -t
  changed_when: false

- name: Restart nginx
  ansible.builtin.systemd:
    name: nginx
    state: restarted
    enabled: true
```

## Variable Precedence Strategy

```yaml
# defaults/main.yml — overridable defaults
app_port: 8080
app_workers: 4
app_log_level: info

# group_vars/prod.yml — environment overrides
app_workers: 16
app_log_level: warn
```

## Conditional Execution

```yaml
- name: Install packages (Debian)
  ansible.builtin.apt:
    name: "{{ debian_packages }}"
  when: ansible_os_family == "Debian"

- name: Install packages (RedHat)
  ansible.builtin.yum:
    name: "{{ redhat_packages }}"
  when: ansible_os_family == "RedHat"
```

## Integration with Terraform

Ansible runs after Terraform provisions infrastructure:

```yaml
# Use dynamic inventory from Terraform outputs
# or AWS EC2 dynamic inventory plugin
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1
filters:
  "tag:Environment": "{{ env }}"
  "tag:Role": "{{ role_filter }}"
keyed_groups:
  - key: tags.Role
    prefix: role
```

## Testing with Molecule

```bash
molecule create     # Create test instances
molecule converge   # Run playbook
molecule verify     # Run tests
molecule destroy    # Cleanup
molecule test       # Full lifecycle
```
