# CiviCRM buildkit on Docker

CiviCRM buildkit on Docker is primarily built for development. It may also be useful for hosting. Contributions welcome.

The docker image at <https://hub.docker.com/r/michaelmcandrew/civicrm/> is built with the [`civicrm/Dockerfile`](civicrm/Dockerfile) in this repo and published nightly. It requires another container running MySQL, such as configured in the `docker-compose.yml`. You may wish to use it with other containers like `phpmyadmin` and `maildev`.

See below for information on [updating your local images](#Upgrading) and an [overview of the architecture](#architecture) of this this container and how to contribute to CiviCRM's cloud-native initiative.

The **docker-compose.yml** file in this repository is a good starting point for a Docker-based CiviCRM development environment. You can replace it with your own, or use an alternate by passing the file location with the `-f` or `--file` options to `docker-compose`. See the section on [Custom Images](#custom-images) below for more tips and how to use the `docker-compose-build.yml` file in this repo.

## Requirements

* Docker
* Docker compose
* WSL2 [for running on Windows]

There is an issue with file permissions if you are using Windows. Follow these [instructions for enabling WSL2](https://docs.docker.com/docker-for-windows/wsl/) integration and upgrading an existing WSL installation to WSL2. Background on the [issue #52](https://github.com/michaelmcandrew/civicrm-buildkit-docker/issues/52).

## Getting started

1. Clone this repository
2. In repository root, change ownership of the build directory to match the user
   you will be running the rest of the command as:

   ```sh
   chown username:usergroup build/
   ```

   See <https://github.com/michaelmcandrew/civicrm-buildkit-docker/issues/63>
3. From the repository root, run `docker-compose up -d` to start the containers
4. Create a dmaster demo site with `docker-compose exec -u buildkit civicrm civibuild create dmaster`

    * The build will run for some time.  If successful, it will conclude with output similar to the following:

        ```
        - CMS_ROOT: /buildkit/build/dmaster
        - CMS_URL: http://dmaster.localhost:7979
        - CMS_DB_DSN: mysql://dmastercms_rftpp:Z8ADa62aJjf6n7IX@mysql:3306/dmastercms_rftpp?new_link=true
        - CIVI_DB_DSN: mysql://dmasterciv_6o62j:zhp2ftwJuw5W4WpG@mysql:3306/dmasterciv_6o62j?new_link=true
        - TEST_DB_DSN: mysql://dmastertes_gdc1v:eGCi5IT2bIkKv4EZ@mysql:3306/dmastertes_gdc1v?new_link=true
        - ADMIN_USER: admin
        - ADMIN_PASS: ##########
        - DEMO_USER: demo
        - DEMO_PASS: demo
        ```

    * Make note of the admin and demo credentials for future use. These credentials may also be found in the build directory, e.g. the credentials for `dmaster` are in `build/dmaster.sh`.
    * For more help with civibuild's create command, see the [buildkit documentation](https://docs.civicrm.org/dev/en/latest/tools/civibuild/)

5. Navigate to your new CiviCRM development site at <http://dmaster.localhost:7979>

In order to 'interact' with your codebase with comand line tools such civix etc, you'll need to run a bash shell 'within' the container. This can be done using the ```bkb``` alias defined below.

Note: for less surprises, consider using a [stable release](https://github.com/michaelmcandrew/civicrm-buildkit-docker/releases).

## Usage

### CLI commands

Buildkit cli commands can be run from the host with:

`docker-compose exec -u buildkit civicrm <COMMAND>`

Alternatively, you can login to the container and run commands from there with:

`docker-compose exec -u buildkit civicrm bash`

Note: to avoid permissions issues, ensure that you connect to the containers as the buildkit user, not as root.

### The `/buildkit/build` mount

The `build` directory of this repository is mounted into the civicrm container at `/buildkit/build`. Builds created in the container are visible here, so files can be edited directly on the host.

### The `/extra` mount

We also mount an the `extra` directory of this repository at `/extra` for ad hoc use.

### Site URLs

By default, new builds created with this Docker image are created at `http://[SITE_NAME].localhost:7979`. This makes it simple to create multiple builds in the same container.

Other URLs can be passed to the civibuild command, however, you will need to ensure any domains are resolved appropriately, e.g. by adding entries to the `/etc/hosts` file.

### phpMyAdmin

phpMyAdmin is available for database admin at <http://localhost:8081>.

### Development email

Navigate to <http://localhost:8082> to view all email sent from the civicrm container.

Background: by default, buildkit disables outbound mail. We delete `civicrm-buildkit/app/civicrm.settings.d/100-mail.php` which re-enables outbound mail. We install `msmtp` on the `civicrm` container and configure it to deliver all mail to the `maildev` container.

### Aliases

docker-compose commands get quite verbose. You may want to create aliases as follows:

```bash
alias bku='docker-compose --file $HOME/civicrm-buildkit-docker/docker-compose.yml up -d'
alias bkc='docker-compose --file $HOME/civicrm-buildkit-docker/docker-compose.yml exec -u buildkit civicrm'
alias bkb='docker-compose --file $HOME/civicrm-buildkit-docker/docker-compose.yml exec -u buildkit civicrm bash'
alias bk='docker-compose --file $HOME/civicrm-buildkit-docker/docker-compose.yml'
```

You can then:

* bring up the containers with `bku`
* run build and admin tools with `bkc`, for example `bkc civibuild create dmaster`.
* start bash in the civicrm container with `bkb`.
* run any other docker compose command with `bk`, e.g. `bk down`

**Note**: the above aliases assume you have downloaded this repo to `$HOME/civicrm-buildkit-docker`. Please adjust as appropriate if you have downloaded it somewhere else.

### Using different versions of PHP

See [publish/README.md](publish/README.md) for instructions on how to build CiviCRM containers that use different versions of PHP.

### UID and GID conflicts

Bind mounts are fussy when it comes to user ids and group ids. If the user on your host machine has a different UID and GID to the buildkit user in the container (which is 1000 by default), you can create a custom build (see below) that passes BUILDKIT_UID and BUILDKIT_GID as arguments.

## Custom Images

This repository comes with a `docker-compose-build.yml` that can be used for custom builds. Custom builds are useful if you want to pass particular arguments to the build. For example, you can define the UID and GID of the buildkit user (see below).

You can create a custom build with a custom UID for the buildkit user as follows:

1. Overwrite `docker-compose.yml` with `docker-compose-build.yml`.
2. Change the BUILDKIT_UID argument to match your user id if necessary (`echo $UID` should give you your user id).
3. run `docker-compose up -d --build`.

## Getting help

Feel free to ask any questions you have in the [docker](https://chat.civicrm.org/civicrm/channels/docker) chatroom (mentioning @michaelmcandrew if you feel like it), or file an issue in the [github issue queue](https://github.com/michaelmcandrew/civicrm-buildkit-docker/issues).

## Upgrading

New docker images are periodically released on Docker Hub at <https://hub.docker.com/r/michaelmcandrew/civicrm/>. You can upgrade to the latest version as follows:

1. Download the latest image with `docker pull michaelmcandrew/civicrm`.
2. Run `docker-compose up -d` will rebuild your containers with the latest image.

**Note**: we mount `/buildkit` as a volume so that any changes you make to the civicrm container (and your bash history, etc.) persist across restarts. This volume is mounted over the `/buildkit` directory that comes with the container.  You will need to delete this volume (it gets recreated automatically based on the contents of the container) if you want to access the /buildkit directory that comes with the upgraded container. You can do that all follows:

From the civicrm-buildkit-docker directory:

1. Stop your containers `docker-compose down`.
2. Identify the /buildkit volume with `docker volume ls`. It will have a 'volume name' along the lines of `civicrmbuildkitdocker_buildkit`.
4. Remove the volume with `docker volume rm [VOLUME NAME]`.
5. Restart the containers with `docker-compose up -d`.

## Architecture

We stick with the defaults and follow best practice whenever possible. Sometimes CiviCRM best practice and Docker best practice are at odds. In these situations we are often forced to do things the CiviCRM way. When this happens, we make a note in the 'cloud native' project of the steps we could take to make CiviCRM more Docker friendly (e.g. environment variables to configure SMTP).

The `docker-compose.yml` defines the following containers:

* **civicrm** (serves the sites; contains build and admin tools like `civibuild`, `cv`, etc.)
* **mysql**
* **phpmyadmin** for easier MySQL admin
* **maildev** to catch outbound email

It seems like might make sense to integrate this repository with [civicrm-buildkit](https://github.com/civicrm/civicrm-buildkit) at some point. I don't think that would be too hard to do.

More general discussion about CiviCRM and Docker is welcome in CivCRM's ['cloud-native' project](https://lab.civicrm.org/dev/cloud-native).

## Credits

CiviCRM Buildkit Docker is written by [Michael McAndrew](https://twitter.com/michaelmcandrew) from [Third Sector Design](https://thirdsectordesign.org/) who you can [contact](https://thirdsectordesign.org/contact) for help, support and further development.

## Contributing

Contributions to this repository are very welcome. Feel free to submit a pull request for minor improvements. For larger changes, please create an issue first.

## See also

* [Publishing Dockerfiles](publish/README.md) for details on how to update the image published on <https://hub.docker.com>.

## License

This extension is licensed under [AGPL-3.0](LICENSE).
