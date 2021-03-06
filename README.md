## Synopsis

The Embedded postgresql maven plugin supports creating of Integration tests in maven by starting up a postgresql instance/process in the pre-integration-test phase of the maven lifecycle and shuts down that same process in the post-integration-phase of the lifecycle. It depends upon the open-source [postgresql-embedded-lib](https://github.com/aramcodz/postgresql-embedded-lib) Java component, which is fork of the [original postresql-embedded project](https://github.com/yandex-qatools/postgresql-embedded).

This plugin has been compiled and tested with Java 1.7 and maven 3.3.9 and PostgreSQL 9.2.4 and PostgreSQL 9.3.6.

## Notes on Usage

Currently (as of 12/01/2016), this plugin only supports the following 9.x versions of [Postgresql](https://www.postgresql.org/):
  * V9_2_4("9.2.4-1")
  * V9_3_6("9.3.6-1")
  * V9_4_4("9.4.4-1")
  * V9_5_0("9.5.0-1")

This restriction comes from the dependency on the [postgresql-embedded](https://github.com/yandex-qatools/postgresql-embedded) Java component.  

You use maven to compile this plugin and install it into your local maven repository (if you download the project or clone this Repo):
  `mvn clean install ` 

An example of using this plugin in the pom for your project (where you wish to use the plugin to set up your Integration tests):
```
      <plugin>
        <groupId>com.smartmonkee.maven</groupId>
        <artifactId>embedded-postgres-maven-plugin</artifactId>
        <version>0.1.0</version>
        <executions>
          <execution>
            <id>manage-service</id>
            <goals>
              <goal>startpostgres</goal>
              <goal>stoppostgres</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <postgresVersion>V9_5_0</postgresVersion>
          <dbName>YOUR_DB_NAMWE</dbName>
          <dbUsername>YOUR_DB_USER_ACCOUNT</dbUsername>
          <dbPassword>YOUR_DB_PASSWORD</dbPassword>
          <proxyUrl>YOUR_PROXY_URL</proxyUrl> 
          <dbPort>5432</dbPort>
          <downloadUrl>YOUR_CUSTOM_POSTGRESQL_DOWNLOAD_URL}</downloadUrl>		  
        </configuration>        
      </plugin>  
``` 

**Plugin Configuration Details:**
The following configuration parameters can be set:
  * skipExecution (boolean)
  * postgresVersion (String, default="V9_3_6")
  * dbURL (String, defaults to "localhost")    
  * dbName (String)
  * dbUsername (String)
  * dbPassword (String)
  * dbPort (Integer - optional, defaults to 5432)  
  * proxyUrl (String - optional)    
  * proxyPort (Integern - optional, defaults to 8080)  
  * downloadUrl (String - optional)

The proxyUrl & proxyPort are both optional and used for accessing the postgreSQL installation download site through a proxy server, if, for instance, your local (corporate) intranet will only allow download through a proxy server.

The downloadUrl is an optional config param and it allows the user to specify a custom web location from which the postgresql-embedded-lib attempts to download and install the correct platorm/version of the PostgreSQL installation ZIP file (e.g. postgresql-9.2.4-1-windows-x64-binaries.zip or postgresql-9.2.4-1-osx-binaries.zip). It defaults to the public site "http://get.enterprisedb.com/postgresql/". But, as I experienced, if you are behind a Corp firewall, you may have some difficulties downloading that file, so you may wish to download the postgres installation files you need yourself, and place them on a local/internal server.

## Known Issues
1. __Note:__ that there appears to be some issue with some confusing/incorrect maven console (Warnings) output when running a maven build that *uses* this plugin. This issue appears to be something caused by the use of the underlying [postgres-embedded Java library](https://github.com/yandex-qatools/postgresql-embedded) So, when the "stop" postgres action runs in the `post-integration-phase` you may see some console output like this:

```
[INFO] --- embedded-postgres-maven-plugin:0.1.0-SNAPSHOT:stoppostgres (manage-service) @ My_User ---
[INFO] Attempting to Stop the Embedded Postgres process
Oct 28, 2016 11:37:16 AM ru.yandex.qatools.embed.postgresql.PostgresProcess stopInternal
INFO: trying to stop postgresql
[INFO] start AbstractPostgresConfig{storage=Storage{dbDir=/var/folders/8g/69wh31fn7nx3q81phwfdpld00000gn/T/postgresql-embed-a304fe66-9b14-47ef-94e8-2de787943696/db-content-589b3260-c83e-4ad2-9742-a135c64de3ac, dbName='myDB', isTmpDir=true}, network=Net{host='localhost', port=5432}, timeout=Timeout{startupTimeout=30000}, credentials=Credentials{username='myDbUser', password='myPassword'}, args=[stop], additionalInitDbParams=[]}
Oct 28, 2016 11:37:17 AM ru.yandex.qatools.embed.postgresql.PostgresProcess sendStopToPostgresqlInstance
INFO: Cleaning up after the embedded process (removing /var/folders/8g/69wh31fn7nx3q81phwfdpld00000gn/T/postgresql-embed-a304fe66-9b14-47ef-94e8-2de787943696)...
Oct 28, 2016 11:37:18 AM ru.yandex.qatools.embed.postgresql.PostgresProcess stopInternal
WARNING: could not stop postgresql with command, try next
[INFO] stopOrDestroyProcess: process hasn't exited 

[INFO] execSuccess: false [kill, -2, 9482]
Oct 28, 2016 11:37:19 AM ru.yandex.qatools.embed.postgresql.PostgresProcess stopInternal
WARNING: could not stop postgresql, try next
[INFO] stopOrDestroyProcess: process hasn't exited 

[INFO] execSuccess: false [kill, 9482]
Oct 28, 2016 11:37:19 AM ru.yandex.qatools.embed.postgresql.PostgresProcess stopInternal
WARNING: could not stop postgresql, try next
Oct 28, 2016 11:37:19 AM ru.yandex.qatools.embed.postgresql.PostgresProcess stopInternal
WARNING: could not stop postgresql the second time, try one last thing
Oct 28, 2016 11:37:19 AM ru.yandex.qatools.embed.postgresql.PostgresProcess deleteTempFiles
WARNING: Could not delete temp db dir: /var/folders/8g/69wh31fn7nx3q81phwfdpld00000gn/T/postgresql-embed-a304fe66-9b14-47ef-94e8-2de787943696/db-content-589b3260-c83e-4ad2-9742-a135c64de3ac
[WARNING] Could not delete pid file: /var/folders/8g/69wh31fn7nx3q81phwfdpld00000gn/T/postgresql-embed-a304fe66-9b14-47ef-94e8-2de787943696/pgsql/bin/postgres.pid
[INFO] Stop of Embedded Postgres process Succeeded!
```
Despite these Warnings & misleading INFO messages, if you see the following log entries, the shutdown postgres process has (very likely) succeded:
```
[INFO] Attempting to Stop the Embedded Postgres process
Oct 28, 2016 11:37:16 AM ru.yandex.qatools.embed.postgresql.PostgresProcess stopInternal
INFO: trying to stop postgresql

INFO: Cleaning up after the embedded process (removing /[SOME_TEMP_MAVEN_FOLDER])...

[INFO] Stop of Embedded Postgres process Succeeded!
```

For reference, [An issue](https://github.com/yandex-qatools/postgresql-embedded/issues/50) has been opened with the maintainers of the postgresql-embedded project.

This issue **does not appear** to affect the running or the Stop process of the postgres embedded Lib, but appears to only be misleading console message information. 

## License

[Apache 2](https://www.apache.org/licenses/LICENSE-2.0)