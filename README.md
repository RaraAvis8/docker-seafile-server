# Docker Image for fully functional Seafile with SQLite

[![Build Status](https://travis-ci.org/stonemaster/docker-seafile-server.svg?branch=master)](https://travis-ci.org/stonemaster/docker-seafile-server)

**Current Seafile server version: 6.0.4**

This docker image provides a fully-functional *Seafile*
server installation that is configured with *SQLite* and
has no other dependencies. Internally *nginx* is configured
as a reverse proxy with can either be accessed via *HTTP*
or *HTTPs*.

 * The container exposes a */data* volume where all
seafile data is stored in a persistent manner
 * Initial setup is done during first startup where
setup of the administator account is created. The administrator
allows to create other users.
 * **No user interaction needed for container startup!**
 * Upgrades to new seafile versions are handled internally
by automatically running seafile provided upgrade scripts
 * Unittests are done using [bats](https://github.com/sstephenson/bats)
 * Automatic Docker builds and Travis builds which run unittests

## Running container

```bash
docker run -d --name seafile_http \
	-p  "80:80" \
	-e SEAFILE_EXTERNAL_PORT=80 \
	-e SEAFILE_HOSTNAME=localhost \
	-e SEAFILE_SERVER_NAME=myseafile \
	-e SEAFILE_ADMIN_MAIL=admin@seafile.com \
	-e SEAFILE_ADMIN_PASSWORD=test123 \
	-v "/tmp/seafile_http:/data" \
	stonemaster/docker-seafile-server
```

## Running container with `docker-compose`

Create a `docker-compose.yml` file:

```
seafile:
    image: stonemaster/docker-seafile-server
    container_name: seafile
    ports:
        - "443:443"
    environment:
        - SEAFILE_EXTERNAL_PORT=443
        - SEAFILE_HOSTNAME=localhost
        - SEAFILE_SERVER_NAME=myseafile
        - SEAFILE_ADMIN_MAIL=admin@seafile.com
        - SEAFILE_ADMIN_PASSWORD=test123
        - USE_SSL=on
    volumes:
        - /tmp/seafile:/data
        - ./test/ssl:/etc/ssl

```

Run `docker-compose up -d`.

## Directory structure for Seafile

The following files are located inside the folder
that is mounted to `/data`:

```
seafile_version
seahub-data
seafile-data
seahub.db
```

`seahub-data` contains the internal seafile file structure
and thus the user data. `seahub.db` contains the SQLite database
used for meta information by Seafile. `seafile_version` is
generated by the container to store the current software
version for seamingless container upgrades. `seafile-data`
contains the actual user files.

## Migration from a previous Seafile installation

**Warning: this is untested!**

 1. Copy the `seahub-data`, `seafile-data` and `seahub.db` directory/files from the existing
installation into your containers to-be-mounted `data` folder.
Don't do that when the container is started!
 2. Create a `seafile_version` file in the `/data` mount containing the current
`MAJOR.MINOR` of your installation e.g. `5.0`. Don't add a
newline after the version.

## SSL

### Generating self-signed certificate

To generate a self-signed certificat using OpenSSL use
this command:

```bash
openssl req -x509 -newkey rsa:4096 -keyout privkey.pem -out cert.pem -nodes -days 365
```

An unprotected private key will be generated using the command
line option `-nodes`. Omit it to generate a password protected
private key file. Mount these two files into the container
to the folder `/etc/ssl` using the default SSL locations.

### Using Let's Encrypt

Mount your letsencrypt folder to `/etc/letsencrypt`. Adapt certificate
and private key filenames using the `SSL_CERT_FILE` and `SSL_PRIVKEY_FILE`
environment variables.

For example add the following parameters to your `docker run` command:

```
   ... -v /etc/letsencrypt:/etc/letsencrypt \
       -e SSL_CERT_FILE=/etc/letsencrypt/live/seafile.com/cert.pem \
       -e SSL_PRIVKEY_FILE=/etc/letsencrypt/live/seafile.com/privkey.pem
```

## Environment variables

### SEAFILE_EXTERNAL_PORT

This port **must** match the port that is exposed
via Docker. It is used internally by Seafile to expose
it to the client for file handling.

### SEAFILE_HOSTNAME

This hostname *must* match the host the Seafile instance
is running on. This hostname is exposed to clients
to allow file handling. If you get errors during file
uploads and downloads, check that `SEAFILE_HOSTNAME` and
`SEAFILE_EXTERNAL_PORT` match your setup.

### SEAFILE_SERVER_NAME

The name of the seafile server instance needed by
Seafile.

### SEAFILE_ADMIN_MAIL

The administator's e-mail used during initial setup.
The administrator is allowed to create other accounts.

### SEAFILE_ADMIN_PASSWORD

The administator's password used during initial setup.

### USE_SSL

 * *default:* `off`
 * Options: `on`, `off`

Enables HTTP (`off`) or HTTPs (`on`).
Adjust `SSL_CERT_FILE` and `SSL_PRIVKEY_FILE`
environment variables if necessary to match your setup.

### SSL_CERT_FILE

 * *default:* `/etc/ssl/cert.pem`

The path to the SSL certificate file.
Ignored when `USE_SSL` equals `off`.

### SSL_PRIVKEY_FILE

 * *default:* `/etc/ssl/privkey.pem`

The path to the SSL private key file.
Ignored when `USE_SSL` equals `off`.

## License

Boost License. See [LICENSE.txt](LICENSE.txt) for more details.
