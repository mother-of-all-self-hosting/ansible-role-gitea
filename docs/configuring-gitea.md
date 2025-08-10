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

This is an [Ansible](https://www.ansible.com/) role which installs [Gitea](https://gitea.org) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Gitea is a self-hosted lightweight software forge (Git hosting service, etc.), an alternative to [Gitea](https://gitea.io/).

See the project's [documentation](https://gitea.org/docs/latest/) to learn what Gitea does and why it might be useful to you.

## Prerequisites

To run a Gitea instance it is necessary to prepare a [Postgres](https://www.postgresql.org) database server.

If you are looking for an Ansible role for it, you can check out [this role (ansible-role-postgres)](https://github.com/mother-of-all-self-hosting/ansible-role-postgres) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

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
gitea_ssh_port: 222
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

## Migrating from Gitea

>[!NOTE]
> The instruction is intended to be applied for Gitea and Gitea instances managed by [the MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook). Replace the service names and paths with the ones specified on your playbook as necessary.

Gitea is a fork of Gitea. Migrating Gitea (versions up to and including v1.22.0) to Gitea was relatively easy, but [Gitea versions after v1.22.0 do not allow such transparent upgrades anymore](https://gitea.org/2024-12-gitea-compatibility/).

Nevertheless, upgrades may be possible with some manual work. Below is a rough guide to help you migrate from Gitea (tested with version v1.23.1) to Gitea (v10.0.0).

> [!WARNING]
> 2FA does not seem to be working in Gitea, but [some work](https://codeberg.org/gitea/gitea/pulls/7251) has landed in [v10.0.2](https://codeberg.org/gitea/gitea/src/commit/c5c0948ae5c96a694cc36945c3d9d5dae439bdd0/release-notes-published/10.0.2.md) which possibly improves things. If this is something you require to get working, do know that it's still an unsolved problem. Details for resetting/disabling 2FA are below.

1. Adjust `vars.yml` to enable the Gitea service (prepare DNS records if using a dedicated hostname instead of your old Gitea hostname). We do not disable Gitea just yet.

2. Stop Gitea by running this on the server: `systemctl disable --now mash-gitea.service`

3. Dump the Gitea database and adapt for Gitea

    Run these commands on the server:

    ```sh
    mkdir /mash/gitea-to-gitea-migration
    chown $(id -u mash):$(id -g mash) /mash/gitea-to-gitea-migration

    docker run \
    --rm \
    --user=$(id -u mash):$(id -g mash) \
    --cap-drop=ALL \
    --env-file=/mash/postgres/env-postgres-psql \
    --mount type=bind,src=/mash/gitea-to-gitea-migration,dst=/out \
    --network=mash-postgres \
    --entrypoint=/bin/sh \
    docker.io/postgres:17.2-alpine \
    -c 'set -o pipefail && pg_dump -h mash-postgres -d gitea > /out/gitea.sql'

    sed --in-place 's/OWNER TO gitea/OWNER TO gitea/g' /mash/gitea-to-gitea-migration/gitea.sql
    ```

4. Prepare the `gitea` Postgres database by running the playbook: `just run-tags install-postgres`

5. Import the database dump into the `gitea` Postgres database

    Run this command on the server:

    ```sh
    docker run \
    --rm \
    --user=$(id -u mash):$(id -g mash) \
    --cap-drop=ALL \
    --env-file=/mash/postgres/env-postgres-psql \
    --mount type=bind,src=/mash/gitea-to-gitea-migration,dst=/out \
    --network=mash-postgres \
    --entrypoint=/bin/sh \
    docker.io/postgres:17.2-alpine \
    -c 'set -o pipefail && psql -h mash-postgres -d gitea < /out/gitea.sql'
    ```

6. Install Gitea using the playbook, but do not start it yet: `just run-tags install-gitea`

7. Sync some files from Gitea by running these commands on the server

    ```sh
    rsync -avr /mash/gitea/data/. /mash/gitea/data/.

    # These files seem to live in a different place now, so we move them around.
    mv /mash/gitea/data/data/gitea/repo-avatars /mash/gitea/data/repo-avatars
    rmdir /mash/gitea/data/data/gitea
    ```

8. **For Gitea versions older than v1.22, skip this step**. For newer versions, revert the `gitea` database to a schema migration version that Gitea supports

    In this guide, we're using Gitea v1.23.1, the database schema version for which (`312`) is newer than what Gitea supports (`305`). We need to [revert all migrations that are newer than `305`](https://github.com/go-gitea/gitea/tree/v1.23.1/models/migrations/v1_23).

    This may not always be possible or easy. For Gitea v1.23.1, the reverts can be done by running `/mash/postgres/bin/cli`, switching to the `gitea` database (`\c gitea`), and running the following queries:

    ```sql
    -- Revert 311
    ALTER TABLE issue DROP COLUMN time_estimate;

    -- Revert 310
    ALTER TABLE protected_branch DROP column priority;

    -- Revert 309
    DROP INDEX IF EXISTS "IDX_notification_u_s_uu";
    DROP INDEX IF EXISTS "IDX_notification_user_id";
    DROP INDEX IF EXISTS "IDX_notification_repo_id";
    DROP INDEX IF EXISTS "IDX_notification_status";
    DROP INDEX IF EXISTS "IDX_notification_source";
    DROP INDEX IF EXISTS "IDX_notification_issue_id";
    DROP INDEX IF EXISTS "IDX_notification_commit_id";
    DROP INDEX IF EXISTS "IDX_notification_updated_by";

    -- Revert 308
    DROP INDEX IF EXISTS "IDX_action_r_u_d";
    DROP INDEX IF EXISTS "IDX_action_au_r_c_u_d";
    DROP INDEX IF EXISTS "IDX_action_c_u_d";
    DROP INDEX IF EXISTS "IDX_action_c_u";

    -- Revert 307
    -- Nothing to do

    -- Revert 306
    ALTER TABLE protected_branch DROP COLUMN block_admin_merge_override;

    -- Mark as reverted
    UPDATE version SET version = 305;
    ```

9. Start Gitea by running this on the server: `systemctl start mash-gitea.service`

10. Complete the Gitea installation on the web

    Gitea may show an Installation page at the base URL with various configuration options, most of which prefilled.

    Our experience was that:

    - the "Run as user" field was empty, but required and "read only". Using the browser's inspector was necessary to remove the `readonly` attribute from the field, so we could enter a `git` value in it

    - the "Disable self-registration" checkbox (which is rightfully checked) had to be disabled to prevent an error saying: "You cannot disable user self-registration without creating an administrator account."

    After completing the installation, you should be able to access your new Gitea instance.

11. Reset 2FA authentication settings for all users

    Gitea seems to suffer from some issues when handling 2FA login and will return a "500 internal server error" while checking your 2FA code. This may be because some secret keys reset during the Gitea -> Gitea migration, or most likely due to some bug.

    If you're experiencing issues with 2FA, run `/mash/postgres/bin/cli`, switch to the `gitea` database (`\c gitea`) and execute the following query:

    ```psql
    DELETE FROM two_factor;
    ```

    After this, you should be able to log in without being prompted for 2FA. Adding 2FA to your account later may be possible, but we also found issues with that:

    > SettingsTwoFactor: Failed to save two factor, pq: invalid byte sequence for encoding "UTF8": ...

    This is probably a bug.

12. If everything is working and you're happy with the migration, consider cleaning up

    You can do this by:

    - removing the Gitea configuration from `vars.yml` and re-running the playbook (e.g. `just run-tags setup-gitea`). This will delete the `/mash/gitea` directory
    - dropping the `gitea` database from the Postgres server (execute `/mash/postgres/bin/cli` and run `DROP DATABASE gitea;`)
    - deleting the `/mash/gitea-to-gitea-migration` temporary directory: `rm -rf /mash/gitea-to-gitea-migration`

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu gitea` (or how you/your playbook named the service, e.g. `mash-gitea`).
