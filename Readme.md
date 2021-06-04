# OpenShift Postgresql / Oracle FDW

This repository can be used to build a container featuring Postgresql, Oracle FDW and optionally the pgcrypto and postgis extensions.

The container features an auto-provisioning step on startup that can be used to establish the connection to Oracle.

## Versions

PostgreSQL versions currently supported are:

- postgresql-9.6

PostGIS versions currently supported are:

- postGIS 2.4

RHEL versions currently supported are:

- RHEL7

CentOS versions currently supported are:

- CentOS7

## Installation

To disable PostGIS, deploy with the environment variable POSTGIS_EXTENSION=N.

To disable pgcrypto, deploy with the environment variable PGCRYPTO_EXTENSION=N.

Otherwise, the deployment will default to `Y`.

### Confirm extensions

Here are the shell commands to confirm the successful creation of the extensions (assuming POSTGIS_EXTENSION and
PGCRYPTO_EXTENSION are set to `Y`):

```
sh-4.2$ psql -d ${POSTGRESQL_DATABASE} -U ${POSTGRESQL_USER} -c "\dx;"
                                     List of installed extensions
   Name     | Version |   Schema   |                             Description
------------+---------+------------+---------------------------------------------------------------------
 oracle_fdw | 1.1     | public     | foreign data wrapper for Oracle access
 pgcrypto   | 1.3     | public     | cryptographic functions
 plpgsql    | 1.0     | pg_catalog | PL/pgSQL procedural language
 postgis    | 2.4.5   | public     | PostGIS geometry, geography, and raster spatial types and functions
(4 rows)
```

Here is the shell command to confirm the verion of PostGIS ((assuming POSTGIS_EXTENSION was set to `Y`):

```
sh-4.2$ psql -d ${POSTGRESQL_DATABASE} -U ${POSTGRESQL_USER} -c "SELECT postgis_full_version();"
                                      postgis_full_version
--------------------------------------------------------------------------------------------------------
 POSTGIS="2.4.5 r16765" PGSQL="96" GEOS="3.6.3-CAPI-1.10.3 80c13047" PROJ="Rel. 4.9.3, 15 August 2016" G
DAL="GDAL 1.11.4, released 2016/01/25" LIBXML="2.9.1" LIBJSON="0.11" RASTER
(1 row)
```

### Choose either the CentOS7 or RHEL7 based image:

RHEL7 based image

```
docker build . -f rhel7.rh-postgresql96/Dockerfile
```

To build the RHEL7 image you need to setup entitlement and subscription manager configurations. In the BC OCP3 cluster this was transparent. In the BC OCP4 cluster this (currently) requires a little extra work. Platform services will have to provision the required RedHat Subscription resources into your build environment.

Upon request (i.e. RocketChat channel), the Platform services team will create a secret (e.g. `platform-services-controlled-etc-pki-entitlement`), and two config maps (e.g. `platform-services-controlled-rhsm-ca`, `platform-services-controlled-rhsm-conf`) into your `-tools` namespace.

Confirm the secret and config maps, inserting them into your customized version of the appropriate `*.sample.bc.*`. For example:

```bash
cp openshift/postgresql96-postgis24-oracle-fdw.sample.bc.yaml openshift/postgresql96-postgis24-oracle-fdw.bc.yaml
```

Once in place, that customized build (e.g. based on [postgresql96-postgis24-oracle-fdw.sample.bc.yaml](./openshift/postgresql96-postgis24-oracle-fdw.sample.bc.yaml) will mount the resources so they are in place for the `rhel7.rh-postgresql96` [Dockerfile](./rhel7.rh-postgresql96/Dockerfile). Additional information can be found here; [Build Entitlements](https://github.com/BCDevOps/OpenShift4-Migration/issues/15#issuecomment-854997077)

CentOS7 based image

```
docker build . -f centos7.rh-postgresql96/Dockerfile
```

## Source

The following open source project was used as a starting point:

https://github.com/sclorg/postgresql-container/tree/master

Refer to the above URL for a reference to the environment variables necessary to configure PostgreSQL.

_NOTE_: This is meant for BC Gov Openshift Builds, as access to internal web servers are required to install the Oracle RPM's.

## FDW Environment Variables

The following environment variables are used to configure the FDW:

| Name               | Purpose                                          | Example                             |
| ------------------ | ------------------------------------------------ | ----------------------------------- |
| FDW_NAME           | The name of the foreign data wrapper             | appname_wrapper                     |
| FDW_FOREIGN_SCHEMA | Oracle schema to get data from                   | oracle_schema                       |
| FDW_FOREIGN_SERVER | The Oracle server reference                      | //servername.domain.name/schemaname |
| FDW_USER           | Oracle username                                  | oracle_username                     |
| FDW_PASS           | Oracle password (store in a secret if sensitive) | **\*\*\*\***                        |
| FDW_SCHEMA         | Postgres schema to send data to                  | oracle_mirror                       |

On startup, this container will do the following:

1. Actions typical of a PostgreSQL container
2. Check for the existance of a flag file in the postgresql data directory.
   1. If it does not exist, do the following:
      1. Check for the FDW environment variables
         1. If they do not exist, show usage instructions to the console log
         2. Otherwise, run the following statements in the Postgresql application database:
            1. Start a transaction
            2. Create the oracle_fdw extension if it does not exist
            3. Drop the FDW server connection if it exists
            4. Create a user mapping that will be used for the FDW server connection
            5. Drop the destination schema. Note that schema is different from database; a database in Postgres can have multiple schemas. The structure will be as follows:
               1. Application Database ->
                  1. public Schema (regular application tables)
                  2. oracle foreign data Schema (copy of the Oracle data)
            6. Create the destination schema
            7. Configure a FDW relationship between Oracle and PostgreSQL
            8. Drop and create a PostgreSQL Role to be used for foreign data access
            9. Grant the role to the Application database user
            10. Update the flag file that will be used to bypass this stage in the future.

## License

Code released under the [Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0).
