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
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up linkding

This is an [Ansible](https://www.ansible.com/) role which installs [linkding](https://codeberg.org/mo8it/linkding) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

linkding is a self-hosted, simple and privacy respecting website traffic tracker. It does not collect IP addresses or browser information. Each visitor is assigned an anonymous ID upon visiting the website, which is used to store information on how long the visitor stays there, without setting cookies.

See the project's [documentation](https://codeberg.org/mo8it/linkding/src/branch/main/README.md) to learn what linkding does and why it might be useful to you.

## Prerequisites

To run a linkding instance it is necessary to prepare a [Postgres](https://www.postgresql.org) database server.

If you are looking for an Ansible role for it, you can check out [this role (ansible-role-postgres)](https://github.com/mother-of-all-self-hosting/ansible-role-postgres) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable linkding with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# linkding                                                             #
#                                                                      #
########################################################################

linkding_enabled: true

########################################################################
#                                                                      #
# /linkding                                                            #
#                                                                      #
########################################################################
```

### Set the hostname

To enable linkding you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
linkding_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting linkding under a subpath (by configuring the `linkding_path_prefix` variable) does not seem to be possible due to linkding's technical limitations.

### Set variables for connecting to a Postgres database server

To have the linkding instance connect to your Postgres server, add the following configuration to your `vars.yml` file.

```yaml
linkding_database_username: YOUR_POSTGRES_SERVER_USERNAME_HERE
linkding_database_password: YOUR_POSTGRES_SERVER_PASSWORD_HERE
linkding_database_hostname: YOUR_POSTGRES_SERVER_HOSTNAME_HERE
linkding_database_port: 5432
linkding_database_name: YOUR_POSTGRES_SERVER_DATABASE_NAME_HERE
```

Make sure to replace values for variables with yours.

### Set the website hostname

You also need to set the hostname of the website, on which the linkding instance counts visits, as below:

```yaml
linkding_tracked_origin: https://origin.example.com
```

Replace `https://origin.example.com` with the hostname of your website.

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `linkding_environment_variables_additional_variables` variable

See the [documentation](https://codeberg.org/mo8it/linkding#configuration) for a complete list of linkding's config options that you could put in `linkding_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, the linkding instance becomes available at the URL specified with `linkding_hostname`. With the configuration above, the service is hosted at `https://example.com`.

To have your linkding instance count visits at `https://origin.example.com`, you need to add the following script tag to the website:

```html
<script type="module" src="https://example.com/count.js"></script>
```

## Troubleshooting

### Check the service's logs

Internal linkding errors will not be logged to `stdout` and will therefore not be part of `journalctl -fu mash-linkding`. Its log can be checked by running `tail -f logs/linkding`.
