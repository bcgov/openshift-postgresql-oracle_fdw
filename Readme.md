# OpenShift Postgresql / Oracle FDW #
This repository can be used to build a container featuring Postgresql 9.5, Oracle FDW

The container features an auto-provisioning step on startup that can be used to establish the connection to Oracle.

## Source ##

The following open source project was used as a starting point:

https://github.com/sclorg/postgresql-container/tree/master/9.5

Refer to the above URL for a reference to the environment variables necessary to configure PostgreSQL.

*NOTE*: This is meant for BC Gov Openshift Builds, as access to internal web servers are required to install the Oracle RPM's.

## FDW Environment Variables ##

The following environment variables are used to configure the FDW:

| Name | Purpose | Example |
| ---- | ------- | ------- |
| FDW_NAME | The name of the foreign data wrapper | appname_wrapper |
| FDW_FOREIGN_SCHEMA | Oracle schema to get data from | oracle_schema |
| FDW_FOREIGN_SERVER | The Oracle server reference |  //servername.domain.name/schemaname |
| FDW_USER | Oracle username | oracle_username |
| FDW_PASS | Oracle password (store in a secret if sensitive) | ******** |
| FDW_SCHEMA | Postgres schema to send data to | oracle_mirror |

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
                5. Drop the destination schema.  Note that schema is different from database; a database in Postgres can have multiple schemas.  The structure will be as follows:
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


