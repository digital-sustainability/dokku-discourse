# dokku-discourse

![Test](https://github.com/digital-sustainability/dokku-discourse/workflows/Test/badge.svg)
![Publish](https://github.com/digital-sustainability/dokku-discourse/workflows/Publish/badge.svg)

Manages discourse apps on your dokku server.

## Plugin installation

```sh
# For dokku >= v0.22.0
dokku plugin:install https://github.com/digital-sustainability/dokku-discourse.git

# For dokku < v0.22.0
dokku plugin:install https://github.com/digital-sustainability/dokku-discourse.git --committish 0.2.1
```

## Plugin update

```sh
dokku plugin:update discourse
```

## Commands

```sh
discourse:help                            # Shows help
discourse:create <app>                    # Creates/rebuilds a discourse app
discourse:destroy <app>                   # Destroys a discourse app
discourse:upgrade <app>                   # Upgrades a discourse app
discourse:install-plugin <app> <git_url>  # Installs a plugin for a discourse app
discourse:list                            # Lists discourse apps
```

## Usage

### Create a new discourse app

#### Standaone

To setup a new discourse app as a separate _standalone_ discourse instance run

```sh
dokku discourse:create discourse-app
```

and choose _false_ in the first prompt.

*A new docker image will be built and this process can take some time.*

You'll be prompted for various discourse configuration values.

#### External Database

To setup a new discourse app with an external database first create a container

```sh
dokku apps:create discourse-app
```

use [dokku-postgres](https://github.com/dokku/dokku-postgres) to create a new postgres service

```sh
dokku postgres:create discourse-db
```

link the container with the postgres service

```sh
dokku postgres:link discourse-db discourse-app
```

this will set an env DATABASE_URL with the following structure:

```sh
postgres://postgres:<DATABSE_PASSWORD>@dokku-postgres-discourse-db:5432/discourse_db
```

you'll be asked to specify a docker link during setup, to get the necessary information run

```sh
dokku docker-options:report discourse-app
```
you should get something like:

```sh
--link dokku.postgres.discourse-db:dokku-postgres-discourse-db 
```

finally start the setup and choose _true_ in the first prompt

```sh
dokku discourse:create discourse-app
```

use the database information from `DATABASE_URL` and `docker-options` to complete the setup.

Data will be stored in location `/var/lib/dokku/data/storage/APP_NAME`.

Once your app is built & deployed, you can treat it as any other dokku app. You should use the [dokku-letsencrypt](https://github.com/dokku/dokku-letsencrypt) plugin for adding TLS.

Continue with the offical [discourse install instructions](https://github.com/discourse/discourse/blob/master/docs/INSTALL-cloud.md#start-discourse) to complete the discourse installation, ignoring any TLS setup instructions.

### Customise the discourse container config

A discourse container config file is created when a discourse app is created. Depeding on your chooise the config is based on [standalone.yml](https://github.com/discourse/discourse_docker/blob/master/samples/standalone.yml) or [web_only](https://github.com/discourse/discourse_docker/blob/master/samples/web_only.yml) including the redis template.

You can edit the config at `/home/dokku/APP_NAME/discourse_standalone.yml` or `/home/dokku/APP_NAME/discourse_web_and_redis.yml`. Don't make any changes to the `volumes` section, but feel free to change anything else. 

*Important:* Changes to the `env` section will overwritten.

After making change be sure to run `dokku discourse:create <app>` to rebuild and re-deploy the discourse app. 

### Upgrade a discourse app

The easiest way to upgrade is to use [the admin](http://discourse.dokku.me/admin/upgrade).

You can also upgrade a discourse app with the following:

```bash
dokku discourse:upgrade discourse-app
```

*The running discourse app will be stopped, the docker image rebuilt and the app redeployed.*

### Add discourse plugins

Install the akismet plugin:

```bash
dokku discourse:install-plugin discourse-app https://github.com/discourse/discourse-akismet
```

*The running discourse app will be stopped, the docker image rebuilt and the app redeployed.*

### Destroy an app

```bash
dokku discourse:destroy discourse-app
```

*Destroying an app does **not** remove the data directory at `/var/lib/dokku/data/storage/APP_NAME`.*

### Uninstall the plugin

```bash
dokku plugin:uninstall discourse
```

*Running discourse apps and images are unaffected.*

To completely remove everything related to discourse:

1. Destroy all discourse apps: `dokku discourse:destroy $(dokku discourse:list -q)`
1. Uninstall the discourse plugin: `dokku plugin:uninstall discourse`
1. Remove the base discourse image: `docker rmi $(docker images -q discourse/base)`
1. Remove discourse data directories at `/var/lib/dokku/data/storage/`.

### Restore from backup

You'll need a backup archive created by discourse, and a fresh discourse app.

Follow the instructions above to create a new discourse app. Ensure you've registered an admin account as you'll need to login to the admin to restore the backup. If the backup site was using TLS, you need to set this up on the new instance before continuing (via `dokku letsencrypt`).

Discourse allows you restore a backup in [the admin](http://discourse.dokku.me/admin/backups), but you need to copy the backup into the discourse container first. This is achieved by copying the backup into the host directory that's mounted to the container.

First, ensure the default backup directory exists on the host:

*Substitute APP_NAME with the name of the discourse app you just created.*

```bash
ssh root@dokku.me
mkdir /var/lib/dokku/data/storage/APP_NAME/backups/default/
```

Exit your server and use scp (or similar) to copy the archive file to the server.

*Substitute APP_NAME with the name of the discourse app you just created.*

```bash
scp discourse-backup.tar.gz root@dokku.me:/var/lib/dokku/data/storage/APP_NAME/backups/default/
```

Login to the admin and restore from backup.

To complete the restoration, you'll need install plugins that were previously installed. Use the `dokku discourse:install-plugin` command to do so.

## Credits

- https://medium.com/batary/deploying-discourse-with-dokku-5eec28e2ad8b
- https://gist.github.com/julienma/a101a72fdd97932bf28909633f45c7be
- https://github.com/badsyntax/dokku-discourse

## Development

See [DEVELOPMENT.md](./DEVELOPMENT.md).

## License

See [LICENSE.md](./LICENSE.md).
