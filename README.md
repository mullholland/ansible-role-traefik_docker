# [Ansible role traefik_docker](#traefik_docker)

Installs and configures traefik container based on official traefik docker container

|GitHub|Downloads|Version|
|------|---------|-------|
|[![github](https://github.com/mullholland/ansible-role-traefik_docker/actions/workflows/molecule.yml/badge.svg)](https://github.com/mullholland/ansible-role-traefik_docker/actions/workflows/molecule.yml)|[![downloads](https://img.shields.io/ansible/role/d/mullholland/traefik_docker)](https://galaxy.ansible.com/mullholland/traefik_docker)|[![Version](https://img.shields.io/github/release/mullholland/ansible-role-traefik_docker.svg)](https://github.com/mullholland/ansible-role-traefik_docker/releases/)|
## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/mullholland/ansible-role-traefik_docker/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---
- name: Converge
  hosts: all
  become: true
  gather_facts: true
  vars:
    traefik_docker_config:
      tls:
        options:
          modern:
            minVersion: "VersionTLS13"
            sniStrict: true

  roles:
    - role: "mullholland.traefik_docker"
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/mullholland/ansible-role-traefik_docker/blob/master/molecule/default/prepare.yml):

```yaml
---
- name: Prepare
  hosts: all
  become: true
  gather_facts: true
  vars:
    pip_packages:
      - "docker"

  roles:
    - role: mullholland.docker
    - role: mullholland.repository_epel
    - role: mullholland.pip
```



## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/mullholland/ansible-role-traefik_docker/blob/master/defaults/main.yml):

```yaml
---
# IMPORTANT
# You should configure the "traefik_docker_ssl_email" to an Email you control
# Default to redirecteing every http (Port 80) traffic to https (Port 443)
# Creates certificates with letsencrypt. So make sure your dns is working
# will automatically create an proxy for each container in the same network (defaults to web)
# the target container needs a specific label set inside the compose file like:
#
# - "traefik.enable=true"
# - "traefik.http.routers.foundry.entryPoints=https"
# - "traefik.http.routers.foundry.rule=Host(`foundry.example.com`)"
# - "traefik.http.routers.foundry.tls.certResolver=letsEncrypt"
# - "traefik.http.routers.foundry.tls=true"

# General config
traefik_docker_network_name: "web"
traefik_docker_base_path: "/opt"
traefik_docker_timezone: "Europe/Berlin"

# User/Group of the stack. Everything is mapped to this, instead of root.
traefik_docker_user: "homelab"
traefik_docker_uid: "900"
traefik_docker_group: "homelab"
traefik_docker_gid: "900"
traefik_docker_users_system: true

# which container version to install
# https://hub.docker.com/_/traefik/tags
# can also be latest
traefik_docker_version: "traefik:latest"

traefik_docker_commands:
  - "--log.level=INFO"
  - "--api.insecure=true"
  - "--api=false"
  - "--accesslog=true"
  - "--providers.docker=true"
  - "--providers.docker.exposedbydefault=false"
  - "--entrypoints.http.address=:80"
  - "--entryPoints.http.http.redirections.entryPoint.to=https"
  - "--entryPoints.http.http.redirections.entryPoint.scheme=https"
  - "--entrypoints.https.address=:443"
  - "--certificatesresolvers.letsEncrypt.acme.httpchallenge=true"
  - "--certificatesresolvers.letsEncrypt.acme.httpchallenge.entrypoint=http"
  - "--certificatesresolvers.letsEncrypt.acme.email={{ traefik_docker_ssl_email }}"
  - "--certificatesresolvers.letsEncrypt.acme.storage=/letsencrypt/acme.json"

# Configure dynamic config
# will be put through `to_nice_yaml`
traefik_docker_config: []
# See https://doc.traefik.io/traefik/https/tls/
# tls:
#   options:
#     modern:
#       minVersion: "VersionTLS13"
#       sniStrict: true

# Email for which to register the letsencrypt emails
traefik_docker_ssl_email: "ssl@example.com"

# additional docker compose environment varaibles
# https://github.com/felddy/foundryvtt-docker?tab=readme-ov-file#optional-variables
traefik_docker_environment_variables: []
traefik_docker_volumes:
  - "/tmp:/tmp"
  - "/var/run/docker.sock:/var/run/docker.sock:ro"
  # dynamic config
  - "/opt/traefik/config/config.yml:/etc/traefik/config.yml"
  - "/opt/traefik/ssl/:/letsencrypt"
# which port to expose. can be empty if used with traefik for example
traefik_docker_ports:
  - "80:80"
  - "443:443"
traefik_docker_labels:
  - "traefik.enable=false"
```

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/mullholland/ansible-role-traefik_docker/blob/master/requirements.txt).

## [State of used roles](#state-of-used-roles)

The following roles are used to prepare a system. You can prepare your system in another way.

| Requirement | GitHub | GitLab |
|-------------|--------|--------|
|[mullholland.repository_epel](https://galaxy.ansible.com/mullholland/repository_epel)|[![Build Status GitHub](https://github.com/mullholland/ansible-role-repository_epel/workflows/Ansible%20Molecule/badge.svg)](https://github.com/mullholland/ansible-role-repository_epel/actions)|[![Build Status GitLab](https://gitlab.com/opensourceunicorn/ansible-role-repository_epel/badges/master/pipeline.svg)](https://gitlab.com/opensourceunicorn/ansible-role-repository_epel)|
|[mullholland.docker](https://galaxy.ansible.com/mullholland/docker)|[![Build Status GitHub](https://github.com/mullholland/ansible-role-docker/workflows/Ansible%20Molecule/badge.svg)](https://github.com/mullholland/ansible-role-docker/actions)|[![Build Status GitLab](https://gitlab.com/opensourceunicorn/ansible-role-docker/badges/master/pipeline.svg)](https://gitlab.com/opensourceunicorn/ansible-role-docker)|
|[mullholland.pip](https://galaxy.ansible.com/mullholland/pip)|[![Build Status GitHub](https://github.com/mullholland/ansible-role-pip/workflows/Ansible%20Molecule/badge.svg)](https://github.com/mullholland/ansible-role-pip/actions)|[![Build Status GitLab](https://gitlab.com/opensourceunicorn/ansible-role-pip/badges/master/pipeline.svg)](https://gitlab.com/opensourceunicorn/ansible-role-pip)|

## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://mullholland.net) for further information.

Here is an overview of related roles:
![dependencies](https://raw.githubusercontent.com/mullholland/ansible-role-traefik_docker/png/requirements.png "Dependencies")

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/mullholland):

|container|tags|
|---------|----|
|[EL](https://hub.docker.com/r/mullholland/enterpriselinux)|all|
|[Fedora](https://hub.docker.com/r/mullholland/fedora/)|38, 39|
|[Ubuntu](https://hub.docker.com/r/mullholland/ubuntu)|all|
|[Debian](https://hub.docker.com/r/mullholland/debian)|all|

The minimum version of Ansible required is 2.10, tests have been done to:

- The previous version.
- The current version.
- The development version.

If you find issues, please register them in [GitHub](https://github.com/mullholland/ansible-role-traefik_docker/issues).

## [License](#license)

[MIT](https://github.com/mullholland/ansible-role-traefik_docker/blob/master/LICENSE).

## [Author Information](#author-information)

[mullholland](https://mullholland.net)
