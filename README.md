# Concept Graph Analytics (CGA)

## Basic deployment

To deploy CGA to a Docker system, use the `deploy.py` tool, which requires Docker Compose.
Required software versions:
- Python 3, version 3.6 or later
- Docker, version 20.10.22 or later
- Docker Compose, version 2.14.1 or later

Log in with your own password to gain access to the Micro Focus IDOL containers on Docker Hub:

```
docker login -u microfocusidolreadonly
```

> To obtain your password (API key) contact Micro Focus support.

Configure the location of your IDOL License Server in `config/base.env`, and grant the `admin` role
in your License Server configuration to the host you will deploy the `analysis` component to.

Add TLS certificates in `config/https/` (see the `Encryption` section).

Run the `deploy.py` tool using Python.  (Much like when running `docker`, you
may have to run it as a different user with sufficient permissions to manage Docker containers.)

```
python3 deploy.py --init auth entity audit cga
```

After the system has started, log into the Swagger UI `localhost:8060/swagger/` with a user that has the `admin` role
and call the POST `/meta/initialize` endpoint to perform a one off initialization.

To show options and other usage information, run:

```
python3 deploy.py --help
```

## Configuration

All configuration files are in the `config` directory.  `base.env` contains settings relevant to
multiple components, and, for example, `api.env` contains settings relevant only to the `api`
component.  Lines starting with `#` are ignored, and these are used to explain the meaning of each
of the settings.

## Encryption

By default, the user-facing servers (authentication server and API) only accept encrypted 
connections.  For this to work, you must obtain TLS certificates and copy them into the `config` 
directory.  The required files are:

- `config/https/api/tls.key`: Private key for the API.
- `config/https/api/tls.crt`: Server certificate for the API.
- `config/https/auth/tls.key`: Private key for the authentication server.
- `config/https/auth/tls.crt`: Server certificate for the authentication server.

## Further examples

To use HTTP instead of HTTPS, for testing purposes only, run:

```
python3 deploy.py --disable-encryption --init auth entity audit cga
```

> note: changes to the encryption state of a deployed system require manual deletion of the realm in Keycloak before running `deploy.py` with the new state.

To resume a stopped CGA system, or to apply changes made to configuration files, or to change which
components are deployed: run the normal command to deploy, but without the `--init` argument:

```
python3 deploy.py auth entity audit cga
```

To stop and remove deployed CGA services, run the Python `deploy.py` tool with no arguments:

```
python3 deploy.py
```

You can deploy components on different hosts, or deploy some components separately using a
compatible implementation (read the `deploy.py` tool help text for a list of components).  For
example, to use an existing object storage server and deploy the audit database on a separate host,
configure hosts and ports in the files in `config/`, and then run on separate hosts (note that
`--init` need only be run once):

```
python3 deploy.py audit
python3 deploy.py --init auth entity cga
```

## System information

By default, the following ports are forwarded ('public' ports listen on all interfaces (0.0.0.0),
while others listen on 127.0.0.1 only):

| **Component** | **Port** | **Public** | **Purpose**                                                          |
|---------------|----------|------------|----------------------------------------------------------------------|
| auth          | 8000     | no         | PostgreSQL database storing authentication server configuration      |
| auth          | 8010     | yes        | Keycloak authentication server (API and admin UI)                    |
| entity        | 8021     | no         | ACI port of IDOL Content database backend for the Gremlin database   |
| entity        | 8022     | no         | Index port of IDOL Content database backend for the Gremlin database |
| entity        | 8023     | no         | Port of Cassandra database backend for the Gremlin database          |
| audit         | 8050     | no         | PostgreSQL database storing audit logs                               |
| api           | 8060     | yes        | System HTTP API                                                      |

Docker volumes are created with the prefix `micro-focus-idol-cga_`, which can be changed using the
`COMPOSE_PROJECT_NAME` setting.  The following volumes are created:

| **Component** | **Volume name**                        | **Purpose**                                      |
|---------------|----------------------------------------|--------------------------------------------------|
| auth          | auth-db-data                           | Authentication server configuration              |
| entity        | entity-storagedb-data                  | Application data                                 |
| entity        | entity-indexdb-data                    | Search index for application data                |
| entity        | entity-indexdb-license-data            | Cache for license information                    |
| audit         | audit-db-data                          | Audit logs                                       |

All containers connect to a Docker network called `micro-focus-idol-cga_main`.  The
`micro-focus-idol-cga` prefix can be changed using the `COMPOSE_PROJECT_NAME` setting.
