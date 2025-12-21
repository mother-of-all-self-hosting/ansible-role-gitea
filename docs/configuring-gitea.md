<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 MASH project contributors
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara
SPDX-FileCopyrightText: 2024 Sergio Durigan Junior

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Gitea

This is an [Ansible](https://www.ansible.com/) role which installs [Gitea](https://gitea.io) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Gitea is a self-hosted lightweight software forge (Git hosting service, etc).

See the project's [documentation](https://docs.gitea.com/) to learn what Gitea does and why it might be useful to you.

## Prerequisites

To run a Gitea instance it is necessary to prepare a [Postgres](https://www.postgresql.org) database server.

If you are looking for an Ansible role for Postgres, you can check out [this role (ansible-role-postgres)](https://github.com/mother-of-all-self-hosting/ansible-role-postgres) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable Gitea with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# gitea                                                                #
#                                                                      #
########################################################################

gitea_enabled: true

########################################################################
#                                                                      #
# /gitea                                                               #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Gitea you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
gitea_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

### Configure SSH port for Gitea (optional)

Gitea uses port 22 for its SSH feature by default.

We recommend you to move your regular SSH server to another port, and stick to this default for your Gitea instance.

If you wish to have the instance listen to another port, add the following configuration to your `vars.yml` file and adjust the port as you see fit.

```yaml
gitea_container_ssh_port: 222
```

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `gitea_environment_variables_additional_variables` variable

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, the Gitea instance becomes available at the URL specified with `gitea_hostname` and `gitea_path_prefix`. With the configuration above, the service is hosted at `https://example.com`.

To get started, open the URL with a web browser, and follow the set up wizard.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu gitea` (or how you/your playbook named the service, e.g. `mash-gitea`).
