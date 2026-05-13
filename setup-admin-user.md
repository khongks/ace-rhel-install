## Setup Administrator User for App Connect Web Console

1. Edit the node.conf.yaml
   ```
   # Admin Security
   #  Authentication
   #  If basicAuth enabled, a maximum of authMaxAttempts authentication attempts are allowed for a client within period authAttemptsDuration
   #  If authMaxAttempts is reached without success, the client is locked out for period authBlockedDuration
   basicAuth: true
   ```

1. Stop the node
   ```
   ibmint stop node INODE01
   ```

1. Eanble admin security
   ```
   mqsichangeproperties INODE01 -b webadmin -n adminSecurity -v active
   mqsichangeproperties INODE01 -b webadmin -n authorizationMode -v file
   ```

1. Setup admin user and password
   ```
   mqsiwebuseradmin INODE01 -c -u admin -a <password> -r adminRole
   ```

1. Start the node
   ```
   ibmint start node INODE01
   ```
