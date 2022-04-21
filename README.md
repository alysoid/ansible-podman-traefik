# [Catena](https://github.com/alysoid/catena) Ansible Role: podman-traefik

Manage [Traefik](https://doc.traefik.io/traefik/) HTTP reverse proxy and load balancer via Ansible using File provider with custom YAML configuration files. The File provider is useful in combination with [Podman](https://wiki.archlinux.org/title/Podman) because Traefik Docker provider is not yet supported (See [#5730](https://github.com/traefik/traefik/issues/5730)).

To manage Rootless containers with Podman and systemd services, check the [ansible-podman-systemd](https://github.com/alysoid/ansible-podman-systemd) role.

## Role variables

### `podman_containers`

List of Podman containers to manage with Traefik when `frontend` is set to `yes`.

```yaml
# Defaults
podman_containers: []

# Example
podman_containers:
  - name: whoami   # Podman container name
    state: created # (created|absent), default is `created`
    frontend: yes  # (yes|no), default is `no`
```

This role requires a file named `{{ item.name }}.yml` into the directory `{{ playbook_dir }}/traefik` which contains Traefik settings relative to the container in YAML format. Here's an example:

```yaml
# `service_ip` is a helper equals to `traefik_server_ip` (see below)
http:
  routers:
    whoami:
      rule: "Host(`whoami.example.com`)"
      service: whoami
      tls: true

  services:
    whoami:
      loadBalancer:
        servers:
          - url: "http://{{ service_ip }}:8081"
```

### `traefik_server_ip`

Server IP address from `inventory_hostname` that is the name for the "current" host being iterated over in the play, `ansible_host` is the IP of the target host. See [Ansible special variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html).

```yaml
# Defaults
traefik_server_ip: "{{ hostvars[inventory_hostname]['ansible_host'] }}"
```

### `traefik_appdata_dir`

Directory for storing Traefik services configuration files.

```yaml
# Defaults
traefik_appdata_dir: "{{ ansible_facts['user_dir'] }}/traefik"
```
