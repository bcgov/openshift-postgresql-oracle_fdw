FROM centos:centos7 

# PostgreSQL image for OpenShift with PostGIS extension.
# Volumes:
#  * /var/lib/psql/data   - Database cluster for PostgreSQL
# Environment:
#  * $POSTGRESQL_USER     - Database user name
#  * $POSTGRESQL_PASSWORD - User's password
#  * $POSTGRESQL_DATABASE - Name of the database to create
#  * $POSTGRESQL_ADMIN_PASSWORD (Optional) - Password for the 'postgres'
#                           PostgreSQL administrative account

ENV POSTGIS_EXTENSION=Y \
    PGCRYPTO_EXTENSION=Y \
    POSTGRESQL_VERSION=9.5 \
    POSTGRESQL_PREV_VERSION=9.4 \
    HOME=/var/lib/pgsql \
    PGUSER=postgres

ENV SUMMARY="PostgreSQL is an advanced Object-Relational database management system" \
    DESCRIPTION="PostgreSQL is an advanced Object-Relational database management system (DBMS). \
The image contains the client and server programs that you'll need to \
create, run, maintain and access a PostgreSQL DBMS server."

LABEL summary=$SUMMARY \
      description="$DESCRIPTION" \
      io.k8s.description="$DESCRIPTION" \
      io.k8s.display-name="PostgreSQL 9.5" \
      io.openshift.expose-services="5432:postgresql" \
      io.openshift.tags="database,postgresql,postgresql95,rh-postgresql95,postgis,postgis24" \
      name="centos/postgresql-95-centos7" \
      com.redhat.component="rh-postgresql95-docker" \
      version="9.5" \
      release="1" \
      maintainer="SoftwareCollections.org <sclorg@redhat.com>"

EXPOSE 5432

COPY root/usr/libexec/fix-permissions /usr/libexec/fix-permissions

# This image must forever use UID 26 for postgres user so our volumes are
# safe in the future. This should *never* change, the last test is there
# to make sure of that.
RUN yum install -y centos-release-scl-rh && \
    INSTALL_PKGS="rsync tar gettext bind-utils rh-postgresql95 rh-postgresql95-postgresql-contrib nss_wrapper rh-postgresql95-postgresql-server rh-postgresql95-postgresql-devel" && \
    yum -y --setopt=tsflags=nodocs install $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum clean all && \
    localedef -f UTF-8 -i en_US en_US.UTF-8 && \
    test "$(id postgres)" = "uid=26(postgres) gid=26(postgres) groups=26(postgres)" && \
    mkdir -p /var/lib/pgsql/data && \
    /usr/libexec/fix-permissions /var/lib/pgsql && \
    /usr/libexec/fix-permissions /var/run/postgresql

# install dev tools used to compile ORACLE_FDW_2_0_0
RUN yum-config-manager --enable rhel-server-rhscl-7-rpms && \
    yum-config-manager --enable rhel-7-server-rpms && \
	yum-config-manager --enable rhel-7-server-eus-rpms && \
    yum-config-manager --enable rhel-7-server-optional-rpms && \
    yum -y groupinstall 'Development Tools' && \
    yum clean all

# install dev tools used to compile ORACLE_FDW_2_0_0
RUN yum-config-manager --enable rhel-server-rhscl-7-rpms && \
    yum-config-manager --enable rhel-7-server-rpms && \
	yum-config-manager --enable rhel-7-server-eus-rpms && \
    yum-config-manager --enable rhel-7-server-optional-rpms && \
	INSTALL_PKGS="wget libaio-devel" && \
    yum -y --setopt=tsflags=nodocs install $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum clean all


# Get prefix path and path to scripts rather than hard-code them in scripts
ENV CONTAINER_SCRIPTS_PATH=/usr/share/container-scripts/postgresql \
    ENABLED_COLLECTIONS=rh-postgresql95

COPY root /

# When bash is started non-interactively, to run a shell script, for example it
# looks for this variable and source the content of this file. This will enable
# the SCL for all scripts without need to do 'scl enable'.
ENV BASH_ENV=${CONTAINER_SCRIPTS_PATH}/scl_enable \
    ENV=${CONTAINER_SCRIPTS_PATH}/scl_enable \
    PROMPT_COMMAND=". ${CONTAINER_SCRIPTS_PATH}/scl_enable"

VOLUME ["/var/lib/pgsql/data", "/var/run/postgresql"]

# install the Oracle dependencies./tmp/oracle_fdw-ORACLE_FDW_2_0_0/oracle_fdw.control
RUN mkdir -p /tmp/oraclelibs && cd /tmp/oraclelibs && \
    wget -nv https://www.pathfinder.gov.bc.ca/filestore/oracle-instantclient12.2-basic-12.2.0.1.0-1.x86_64.rpm && \
    wget -nv https://www.pathfinder.gov.bc.ca/filestore/oracle-instantclient12.2-devel-12.2.0.1.0-1.x86_64.rpm && \
    rpm -Uvh oracle-instantclient12.2-basic-12.2.0.1.0-1.x86_64.rpm && \
    rpm -Uvh oracle-instantclient12.2-devel-12.2.0.1.0-1.x86_64.rpm

COPY root /

ENV PGCONFIG /opt/rh/rh-postgresql95/root/usr/bin
ENV ORACLE_HOME /usr/lib/oracle/12.2/client64/lib
ENV PATH /opt/rh/rh-postgresql95/root/usr/bin/:${PATH}

# aquire and build ORACLE_FDW_2_0_0
RUN cd /tmp && \
    wget -nv https://github.com/laurenz/oracle_fdw/archive/ORACLE_FDW_2_0_0.tar.gz && \
    tar -xzf ORACLE_FDW_2_0_0.tar.gz && \
    cd oracle_fdw-ORACLE_FDW_2_0_0   && \
    make && \
    make install && \
    rm -rf /tmp/oraclelibs /tmp/ORACLE_FDW_2_0_0.tar.gz /tmp/oracle_fdw-ORACLE_FDW_2_0_0 /var/cache/yum

# Aquire and build PostGIS 2.4, for PostgreSQL 9.5
RUN cd /tmp && \
    rpm -ivh https://download.postgresql.org/pub/repos/yum/9.5/redhat/rhel-7-x86_64/pgdg-centos95-9.5-3.noarch.rpm && \
    yum install -y epel-release && \
    yum install -y postgis24_95 postgis24_95-client && \
    /usr/bin/install -c -m 755  /usr/pgsql-9.5/lib/postgis* '/opt/rh/rh-postgresql95/root/usr/lib64/pgsql/' && \
    /usr/bin/install -c -m 755  /usr/pgsql-9.5/lib/rtpostgis* '/opt/rh/rh-postgresql95/root/usr/lib64/pgsql/' && \
    /usr/bin/install -c -m 644 /usr/pgsql-9.5/share/extension/postgis* '/opt/rh/rh-postgresql95/root/usr/share/pgsql/extension/' && \
    mv /usr/pgsql-9.5/share/contrib/postgis-2.4/ /opt/rh/rh-postgresql95/root/usr/share/pgsql/contrib/ && \
    rm -rf /tmp/pgdg-centos95-9.5-3.noarch.rpm /var/cache/yum 

# set the oracle library path
ENV LD_LIBRARY_PATH /usr/lib/oracle/12.2/client64/lib:${LD_LIBRARY_PATH}
USER 26

ENTRYPOINT ["container-entrypoint"]
CMD ["run-postgresql"]
