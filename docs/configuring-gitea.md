<!--
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2020-2024 MDAD project contributors
SPDX-FileCopyrightText: 2020-2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 MASH project contributors
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 Sergio Durigan Junior
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Gitea

This is an [Ansible](https://www.ansible.com/) role which installs [Gitea](https://gitea.io) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Gitea is a self-hosted lightweight software forge (Git hosting service, etc).

See the project's [documentation](https://docs.gitea.com/) to learn what Gitea does and why it might be useful to you.

## Prerequisites

To run a Gitea instance it is necessary to prepare a database. You can use a [MySQL](https://www.mysql.com/) compatible database server, [Postgres](https://www.postgresql.org/), or [SQLite](https://www.sqlite.org/).

If you are looking for Ansible roles for a MySQL compatible server or Postgres, you can check out [ansible-role-mariadb](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) and [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres), both of which are maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

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

### Specify database

It is necessary to select database used by Gitea from a MySQL compatible database, Postgres, and SQLite.

To use Postgres, add the following configuration to your `vars.yml` file:

```yaml
gitea_database_type: postgres
```

Set `mysql` to use a MySQL compatible database and `sqlite` to use SQLite, respectively. The SQLite database is stored in the directory specified with `gitea_data_path`.

For other settings, check variables such as `gitea_database_*` on [`defaults/main.yml`](../defaults/main.yml).

### Configure SSH port for Gitea (optional)

Gitea uses port 22 for its SSH feature by default.

We recommend you to move your regular SSH server to another port, and stick to this default for your Gitea instance.

If you wish to have the instance listen to another port, add the following configuration to your `vars.yml` file and adjust the port as you see fit.

```yaml
gitea_container_ssh_port: 222
```

### Configuring cache (optional)

Gitea uses caching to avoid repeating expensive operations. By default the internal memory (`memory`) is enabled for it, but you can use a specific cache adapter by adding the following configuration to your `vars.yml` file (adapt to your needs):

```yaml
# Valid values: memcache, memory, redis, redis-cluster, twoqueue
gitea_config_cache_adapter: ""
```

#### Configuration for `redis` and `redis-cluster`

If `redis` or `redis-cluster` is set to `gitea_config_cache_adapter`, you can set the connection string by adding the following configuration to your `vars.yml` file:

```yaml
gitea_redis_hostname: YOUR_REDIS_SERVER_HOSTNAME_HERE
gitea_redis_port: 6379
gitea_redis_password: YOUR_REDIS_SERVER_PASSWORD_HERE
gitea_redis_database: 0
```

Make sure to replace `YOUR_REDIS_SERVER_HOSTNAME_HERE` and `YOUR_REDIS_SERVER_PASSWORD_HERE` with your own values.

If you are looking for an Ansible role for Redis, you can check out [ansible-role-redis](https://github.com/mother-of-all-self-hosting/ansible-role-redis) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team. The roles for [KeyDB](https://keydb.dev/) ([ansible-role-keydb](https://github.com/mother-of-all-self-hosting/ansible-role-keydb)) and [Valkey](https://valkey.io/) ([ansible-role-valkey](https://github.com/mother-of-all-self-hosting/ansible-role-valkey)) are available as well.

#### Configuration for `memcache` and `twoqueue`

If `memcache` or `twoqueue` is set to `gitea_config_cache_adapter`, you can set the connection string by adding the following configuration to your `vars.yml` file:

```yaml
gitea_config_cache_host: CONNECTION_STRING_HERE
```

See [this section](https://docs.gitea.com/administration/config-cheat-sheet#cache-cache) on the official documentation for details.

If you are looking for an Ansible role for [Memcached](https://memcached.org), you can check out [ansible-role-memcache](https://app.radicle.xyz/nodes/seed.radicle.garden/rad%3Az2arPcue4GZ6G6FY3gZexsJXqHyDs) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

### Configuring issue indexer (optional)

By default Gitea is configured to use `bleve` as an issue indexer, but you can use a specific indexer by adding the following configuration to your `vars.yml` file (adapt to your needs):

```yaml
# Valid values: bleve, db, elasticsearch, meilisearch
gitea_environment_variables_indexer_issue_indexer_type: ISSUE_INDEXER_VALUE_HERE

# Specify issue indexer connection string
gitea_environment_variables_indexer_issue_indexer_conn_str: YOUR_ISSUE_INDEXER_CONNECTION_STRING_HERE
```

See [this section](https://docs.gitea.com/administration/config-cheat-sheet#indexer-indexer) on the official documentation for details.

>[!NOTE]
> The default Admin API Key is sufficient for using Meilisearch on a Gitea instance. It is [not recommended](https://www.meilisearch.com/docs/learn/security/basic_security) to use the master key for operations anything but managing other API keys.

If you are looking for an Ansible role for Meilisearch, you can check out [ansible-role-meilisearch](https://github.com/mother-of-all-self-hosting/ansible-role-meilisearch) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

### Extending the configuration

There are some additional things you may wish to configure about the service.

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
