# Install and configure PostgreSQL on RHEL for App Connect

## PostgreSQL

### Install PostgreSQL on RHEL/Rocky v10

1. Add Official PostgreSQL Repository.
   ```
   dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-10-x86_64/pgdg-redhat-repo-latest.noarch.rpm
   ```

1. Disable Default PostgreSQL Module.

   ```
   sudo dnf -qy module disable postgresql
   ```

1. Installs PostgreSQL 17 server and additional contrib tools.
   ```
   sudo dnf install -y postgresql17-server postgresql17-contrib
   ```

1. Initialize Database Cluster.
   ```
   sudo /usr/pgsql-17/bin/postgresql-17-setup initdb
   ```
   (Optional) Clear database before re-initialize database cluster.
   ```
   sudo rm -rf /var/lib/pgsql/17/data/*
   ```

1. Ensure correct directory ownership.
   ```
   sudo chown -R postgres:postgres /var/lib/pgsql/17/data
   sudo chmod 700 /var/lib/pgsql/17/data
   ```

1. Enable and Start Service.
   ```
   sudo systemctl enable --now postgresql-17
   ```
   Start, if needed
   ```
   sudo systemctl start postgresql-17
   ```
   Restart, if needed
   ```
   sudo systemctl restart postgresql-17
   ```

1. Verify status of service.
   ```
   sudo systemctl status postgresql-17
   ```

1. Access Database CLI.
   ```
   sudo -u postgres psql
   ```

### Reset password for `postgress` admin user

1. Set a Password for the Admin User (in postgres=#).
   ```
   ALTER USER postgres WITH PASSWORD 'your_secure_password';
   \q
   ```

### Create application database for ACE `acedb`

1. Create a Database and Application User (in postgres=#).
   ```
   CREATE USER aceuser WITH PASSWORD 'user_password';
   CREATE DATABASE acedb OWNER aceuser;
   GRANT ALL PRIVILEGES ON DATABASE acedb TO aceuser;
   \q
   ```

### Allow remote TCP connections

1. Edit `/var/lib/pgsql/17/data/postgresql.conf`
   ```
   listen_addresses = '*'
   ```

1. Edit `/var/lib/pgsql/17/data/pg_hba.conf`
   ```
   # TYPE  DATABASE        USER            ADDRESS                 METHOD
   host    all             all             0.0.0.0/0               scram-sha-256
   ```

1. Restart postgresql service.
   ```
   sudo systemctl restart postgresql-17
   ```
   
1. Verify port is listening.
   ```
   ss -tlpn | grep 5432
   ```

1. If firewalld is active, open the port to remote connection.
   ```
    # Open PostgreSQL service port (5432)
    sudo firewall-cmd --add-service=postgresql --permanent

    # Reload firewall rules to apply changes
    sudo firewall-cmd --reload
   ```

1. Test connection from a remote computer.
   ```
   psql -h <hostname> -p 5432 -U aceuser -d acedb
   ```

### Manage Database Cheatsheet

1. List all databases
   ```
   \l
   ```

1. Connect / switch to database
   ```
   \c <dbname>
   ```

1. List tables
   ```
   \dt
   ```

1. List all users and their privileges
   ```
   \du
   ```

1. Show connection information
   ```
   \conninfo
   ```

1. Create table
   ```
   CREATE TABLE accounts (
      account_id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      account_holder_name VARCHAR(255) NOT NULL,
      account_type       VARCHAR(20)  NOT NULL CHECK (account_type IN ('CHECKING','SAVINGS')),
      balance            DECIMAL(15,2) NOT NULL DEFAULT 0.00,
      currency           CHAR(3)      NOT NULL DEFAULT 'USD',
      created_at         TIMESTAMP    NOT NULL DEFAULT NOW(),
      updated_at         TIMESTAMP    NOT NULL DEFAULT NOW()
   );
   ```

1. Show table schema
   ```
   \d accounts
   ```

1. Insert row
   ```
   INSERT INTO employees (name, email, department) VALUES ('Alice Smith', 'alice@example.com', 'Engineering'), ('Bob Jones', 'bob@example.com', 'Marketing');
   ```

1. Select rows
   ```
   SELECT * FROM accounts;
   ```

1. Drop table
   ```
   SELECT * FROM accounts;
   ```

1. Delete table data
   ```
   TRUNCATE TABLE accounts;
   ```

1. Count rows.
   ```
   SELECT COUNT(*) FROM accounts;
   ```

## App Connect Enterprise - ODBC configuration

1. Copy INI sample files from `/opt/ibm/ace-13.0.9.0/server/ODBC/unixodbc` to a designated folder (e.g. `/var/mqsi/odbc`).
   ```
   cp /opt/ibm/ace-13.0.9.0/server/ODBC/unixodbc/odbc.ini /var/mqsi/odbc
   cp /opt/ibm/ace-13.0.9.0/server/ODBC/unixodbc/odbcinst.ini /var/mqsi/odbc
   ```
   where `/opt/ibm/ace-13.0.9.0` is the installed ACE folder.

1. Set group ownership of the files.
   ```
   chgrp mqbrkrs /var/mqsi/odbc/odbc.ini
   chgrp mqbrkrs /var/mqsi/odbc/odbcinst.ini

   cdmod 644 /var/mqsi/odbc/odbc.ini
   cdmod 644 /var/mqsi/odbc/odbcinst.ini
   ```

1. Edit `odbc.ini`.
   ```
   [ODBC Data Sources]
   MY_PG_DSN=DataDirect ODBC PostgreSQL Wire Protocol

   [MY_PG_DSN]
   Driver=/opt/ibm/ace-13.0.9.0/server/ODBC/drivers/lib/UKpsql95.so
   Description=DataDirect ODBC PostgreSQL Wire Protocol
   Database=acedb
   HostName={{hostname}}
   PortNumber=5432

   [ODBC]
   InstallDir=/opt/ibm/ace-13.0.9.0/server/ODBC/drivers
   UseCursorLib=0
   IANAAppCodePage=4
   UNICODE=UTF-8
   ```

1. Edit `odbcinst.ini`.
   ```
   [ODBC]
   ;# To turn on ODBC trace set Trace=yes
   Trace=no
   TraceFile=/var/log/odbctrace.out
   Threading=2
   ```

1. Export environment variables.
   ```
   export ODBCINI=/var/mqsi/odbc/odbc.ini
   export ODBCSYSINI=/var/mqsi/odbc
   ```

1. Set ODBC credentials to access Database `acedb`.
   ```
   mqsisetdbparms INODE01 -n odbc::MY_PG_DSN -u aceuser -p {{password}}
   ```

1. Report credentials for ODBC credentials
   ```
   mqsireportdbparms  INODE01 -n odbc::MY_PG_DSN
   ```
   Results:
   ```
   BIP8180I: The resource name 'odbc::MY_PG_DSN' has userID 'aceuser'.
   ```

1. Test configuration.
   ```
   mqsicvp INODE01 -n MY_PG_DSN
   ```
   Result:
   ```
   BIP8299I: User 'aceuser' from security resource name 'odbc::MY_PG_DSN' will be used for the connection to datasource 'MY_PG_DSN'. 
   BIP8290I: Verification passed for the ODBC environment. 

   BIP8270I: Connected to Datasource 'MY_PG_DSN' as user 'aceuser'. The datasource platform is 'PostgreSQL', version '17.11.0000 PostgreSQL 17.11'. 
   ===========================
   databaseProviderVersion      = 17.11.0000 PostgreSQL 17.11
   driverVersion                = 08.02.3884 (B4197, U4313)
   driverODBCVersion            = 03.52
   driverManagerVersion         = 03.52.0002.0003
   driverManagerODBCVersion     = 03.52
   databaseProviderName         = PostgreSQL
   datasourceServerName         = acevm.dev.fyre.ibm.com
   databaseName                 = acedb
   odbcDatasourceName           = MY_PG_DSN
   driverName                   = UKpsql95.so
   supportsStoredProcedures     = Yes
   procedureTerm                = procedure
   accessibleTables             = Yes
   accessibleProcedures         = No
   identifierQuote              = "
   specialCharacters            = None
   describeParameter            = Yes
   schemaTerm                   = schema
   tableTerm                    = table
   sqlSubqueries                = 31
   activeEnvironments           = 0
   maxDriverConnections         = 0
   maxCatalogNameLength         = 0
   maxColumnNameLength          = 63
   maxSchemaNameLength          = 63
   maxStatementLength           = 0
   maxTableNameLength           = 63
   supportsDecimalType          = Yes
   supportsDateType             = No
   supportsTimeType             = No
   supportsTimeStampType        = No
   supportsIntervalType         = No
   supportsAbsFunction          = Yes
   supportsAcosFunction         = Yes
   supportsAsinFunction         = Yes
   supportsAtanFunction         = Yes
   supportsAtan2Function        = Yes
   supportsCeilingFunction      = Yes
   supportsCosFunction          = Yes
   supportsCotFunction          = Yes
   supportsDegreesFunction      = Yes
   supportsExpFunction          = Yes
   supportsFloorFunction        = Yes
   supportsLogFunction          = Yes
   supportsLog10Function        = Yes
   supportsModFunction          = Yes
   supportsPiFunction           = Yes
   supportsPowerFunction        = Yes
   supportsRadiansFunction      = Yes
   supportsRandFunction         = No
   supportsRoundFunction        = Yes
   supportsSignFunction         = Yes
   supportsSinFunction          = Yes
   supportsSqrtFunction         = Yes
   supportsTanFunction          = Yes
   supportsTruncateFunction     = Yes
   supportsConcatFunction       = Yes
   supportsInsertFunction       = Yes
   supportsLcaseFunction        = Yes
   supportsLeftFunction         = Yes
   supportsLengthFunction       = Yes
   supportsLTrimFunction        = Yes
   supportsPositionFunction     = Yes
   supportsRepeatFunction       = Yes
   supportsReplaceFunction      = Yes
   supportsRightFunction        = Yes
   supportsRTrimFunction        = Yes
   supportsSpaceFunction        = Yes
   supportsSubstringFunction    = Yes
   supportsUcaseFunction        = Yes
   supportsExtractFunction      = Yes
   supportsCaseExpression       = Yes
   supportsCastFunction         = Yes
   supportsCoalesceFunction     = Yes
   supportsNullIfFunction       = Yes
   supportsConvertFunction      = Yes
   supportsSumFunction          = Yes
   supportsMaxFunction          = Yes
   supportsMinFunction          = Yes
   supportsCountFunction        = Yes
   supportsBetweenPredicate     = Yes
   supportsExistsPredicate      = Yes
   supportsInPredicate          = Yes
   supportsLikePredicate        = Yes
   supportsNullPredicate        = Yes
   supportsNotNullPredicate     = Yes
   supportsLikeEscapeClause     = Yes
   supportsClobType             = No
   supportsBlobType             = No
   charDatatypeName             = character
   varCharDatatypeName          = character varying
   longVarCharDatatypeName      = text
   clobDatatypeName             = N/A
   timeStampDatatypeName        = N/A
   binaryDatatypeName           = bit
   varBinaryDatatypeName        = bit varying
   longVarBinaryDatatypeName    = bytea
   blobDatatypeName             = N/A
   intDatatypeName              = integer
   doubleDatatypeName           = double precision
   varCharMaxLength             = 0
   longVarCharMaxLength         = 0
   clobMaxLength                = 0
   varBinaryMaxLength           = 0
   longVarBinaryMaxLength       = 0
   blobMaxLength                = 0
   timeStampMaxLength           = 0
   identifierCase               = Lower
   escapeCharacter              = \
   longVarCharDatatype          = -1
   clobDatatype                 = 0
   longVarBinaryDatatype        = -4
   blobDatatype                 = 0

   BIP8273I: The following datatypes and functions are not natively supported by datasource 'MY_PG_DSN' using this ODBC driver: Unsupported datatypes: 'DATE, TIME, TIMESTAMP, INTERVAL, CLOB, BLOB' Unsupported functions: 'RAND' 
   Examine the specific datatypes and functions not supported natively by this datasource using this ODBC driver.  
   When using these datatypes and functions within ESQL, the associated data processing is done within IBM App Connect Enterprise rather than being processed by the database provider.  
   
   Note that "functions" within this message can refer to functions or predicates. 


   BIP8071I: Successful command completion. 
   ```

1. Restart integration node.

## App Connect Enterprise - Application





####  Reference:

1. [How to Connect PostgreSQL Database using IBM App Connect Enterprise](https://community.ibm.com/community/user/blogs/p-venkata-subba-reddy/2024/02/10/ibm-app-connect-enterprise-with-postgresql)
1. [Connecting to PostgreSQL from App Connect Enterprise using the ODBC driver on Linux](https://community.ibm.com/community/user/blogs/srecko-janjic/2023/04/13/connecting-app-connect-enterprise-to-postgresql-us)
