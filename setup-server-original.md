These are the original steps from (approximately, 2.31.1) [9.4 Server setup](https://docs.dhis2.org/master/en/implementer/html/install_server_setup.html).

1. Creating a user to run DHIS2

    ```shell
    sudo useradd -d /home/dhis -m dhis -s /bin/false
    sudo passwd dhis
    ```

2. Creating the configuration directory

    ```shell
    mkdir /home/dhis/config
    chown dhis:dhis /home/dhis/config
    ```

3. Setting server time zone and locale

    ```shell
    sudo dpkg-reconfigure tzdata
    locale -a
    sudo locale-gen nb_NO.UTF-8
    ```

4. PostgreSQL installation

    ```shell
    sudo apt-get install postgresql-10 postgresql-contrib-10 postgresql-10-postgis-2.4
    sudo -u postgres createuser -SDRP dhis
    sudo -u postgres createdb -O dhis dhis2
    ```

    > Return to your session by invoking `exit`.

    > If the DHIS 2 database user does not have permission to create extensions you can create \[PostGIS] from the console using the postgres user with the following commands:

    ```shell
    sudo -u postgres psql -c "create extension postgis;" dhis
    ```

    > Exit the console and return to your previous user with `\q` followed by `exit`.

5. PostgreSQL performance tuning

    > Tuning PostgreSQL is necessary to achieve a high-performing system but is optional in terms of getting DHIS2 to run. PostgreSQL is configured and tuned through the `postgresql.conf` file which can be edited like this:

    ```shell
    sudo nano /etc/postgresql/10/main/postgresql.conf
    ```

    and set the following properties:

    ```properties
    max_connections = 200
    ```

    > Determines maximum number of connections which PostgreSQL will allow.

    ```properties
    shared_buffers = 3200MB
    ```

    > Determines how much memory should be allocated exclusively for PostgreSQL caching. This setting controls the size of the kernel shared memory which should be reserved for PostgreSQL. Should be set to around 40% of total memory dedicated for PostgreSQL.

    ```properties
    work_mem = 20MB
    ```

    > Determines the amount of memory used for internal sort and hash operations. This setting is per connection, per query so a lot of memory may be consumed if raising this too high. Setting this value correctly is essential for DHIS2 aggregation performance.

    ```properties
    maintenance_work_mem = 512MB
    ```

    > Determines the amount of memory PostgreSQL can use for maintenance operations such as creating indexes, running vacuum, adding foreign keys. Incresing this value might improve performance of index creation during the analytics generation processes.

    ```properties
    effective_cache_size = 8000MB
    ```

    > An estimate of how much memory is available for disk caching by the operating system (not an allocation) and is used by PostgreSQL to determine whether a query plan will fit into memory or not. Setting it to a higher value than what is really available will result in poor performance. This value should be inclusive of the `shared_buffers` setting. PostgreSQL has two layers of caching: The first layer uses the kernel shared memory and is controlled by the `shared_buffers` setting. PostgreSQL delegates the second layer to the operating system disk cache and the size of available memory can be given with the `effective_cache_size` setting.

    ```properties
    checkpoint_completion_target = 0.8
    ```

    > Sets the memory used for buffering during the WAL write process. Increasing this value might improve throughput in write-heavy systems.

    ```properties
    synchronous_commit = off
    ```

    > Specifies whether transaction commits will wait for WAL records to be written to the disk before returning to the client or not. Setting this to `off` will improve performance considerably. It also implies that there is a slight delay between the transaction is reported successful to the client and it actually being safe, but the database state cannot be corrupted and this is a good alternative for performance-intensive and write-heavy systems like DHIS2.

    ```properties
    wal_writer_delay = 10000ms
    ```

    > Specifies the delay between WAL write operations. Setting this to a high value will improve performance on write-heavy systems since potentially many write operations can be executed within a single flush to disk.

    ```properties
    random_page_cost = 1.1
    ```

    > _SSD only._ Sets the query planner’s estimate of the cost of a non-sequentially-fetched disk page. A low value will cause the system to prefer index scans over sequential scans. A low value makes sense for databases running on SSDs or being heavily cached in memory. The default value is `4.0` which is reasonable for traditional disks.

    ```properties
    max_locks_per_transaction = 96
    ```

    > Specifies the average number of object locks allocated for each transaction. This is set mainly to allow upgrade routines which touch a large number of tables to complete.

    > Restart PostgreSQL by invoking `sudo /etc/init.d/postgresql restart`

6. Database configuration

    > The database connection information is provided to DHIS2 through a configuration file called `dhis.conf`. Create this file and save it in the `DHIS2_HOME` directory. As an example this location could be:

    ```
    sudo -u dhis nano /home/dhis/config/dhis.conf
    ```

    > A configuration file for PostgreSQL corresponding to the above setup has these properties:

    ```properties
    # Hibernate SQL dialect
    connection.dialect = org.hibernate.dialect.PostgreSQLDialect

    # JDBC driver class
    connection.driver_class = org.postgresql.Driver

    # Database connection URL
    connection.url = jdbc:postgresql:dhis2

    # Database username
    connection.username = dhis

    # Database password
    connection.password = xxxx

    # Database schema behavior, can be validate, update, create, create-drop
    connection.schema = update

    # Encryption password (sensitive)
    encryption.password = xxxx
    ```

    > The `encryption.password` property is the password used when encrypting and decrypting data in the database. Note that the password must not be changed once it has been set and data has been encrypted as the data can then no longer be decrypted. Remember to set a strong password of at least 24 characters .
    >
    > Note that the configuration file supports environment variables. This means that you can set certain properties as environment variables and have them resolved by DHIS 2, e.g. like this where `DB_PASSWD` is the name of the environment variable:

    ```
    connection.password = ${DB_PASSWD}
    ```

    > A common mistake is to have a white-space after the last property value so make sure there is no white-space at the end of any line. Also remember that this file contains the clear text password for your DHIS2 database so it needs to be protected from unauthorized access. To do this invoke the following command which ensures that only the `dhis` user which owns the file is allowed to read it:

    ```
    chmod 0600 dhis.conf
    ```

7. Java installation

    > Oracle Java 8 JDK is the recommended Java option as it provides the greatest operating system support including Ubuntu LTS 14.04. The _webupd8team_ Java PPA provides the necessary packages.

    ```
    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get update
    sudo apt-get install oracle-java8-installer
    ```

    > Check that your installation is okay by invoking:

    ```
    java -version
    ```

    > You can also ensure that the appropriate environment variables are set by installing this package:

    ```
    sudo apt-get install oracle-java8-set-default
    ```

8. Tomcat and DHIS2 installation

    > To install the Tomcat servlet container we will utilize the Tomcat user package by invoking:

    ```
    sudo apt-get install tomcat8-user
    ```

    > This package lets us easily create a new Tomcat instance. The instance will be created in the current directory. An appropriate location is the home directory of the `dhis` user:

    ```
    cd /home/dhis/
    sudo tomcat8-instance-create tomcat-dhis
    sudo chown -R test:test test-dhis/
    ```

    > This will create an instance in a directory called `tomcat-dhis`. Note that the `tomcat7-user` package allows for creating any number of _dhis_ instances if that is desired.

    > Next edit the file `tomcat-dhis/bin/setenv.sh` and add the lines below. The first line will set the location of your Java Runtime Environment, the second will dedicate memory to Tomcat and the third will set the location for where DHIS2 will search for the `dhis.conf` configuration file. Please check that the path the Java binaries are correct as they might vary from system to system, e.g. on AMD systems you might see `/java-8-openjdk-amd64` Note that you should adjust this to your environment:

    ```
    export JAVA_HOME='/usr/lib/jvm/java-8-oracle/'
    export JAVA_OPTS='-Xmx7500m -Xms4000m'
    export DHIS2_HOME='/home/dhis/config'
    ```

    > The Tomcat configiration file is located in `tomcat-dhis/conf/server.xml`. The element which defines the connection to DHIS is the `Connector` element with port 8080. You can change the port number in the `Connector` element to a desired port if necessary. If UTF-8 encoding of request data is needed, make sure that the `URIEncoding` attribute is set to `UTF-8`.

    ```
    <Connector port="8080" protocol="HTTP/1.1"
    connectionTimeout="20000"
    redirectPort="8443"
    URIEncoding="UTF-8" />
    ```

    > The next step is to download the DHIS2 WAR file and place it into the `webapps` directory of Tomcat. You can download the DHIS2 version 2.30 WAR release like this (replace `2.30` with your preferred version if necessary):

    ```
    wget https://releases.dhis2.org/2.30/dhis.war
    ```

    > _Note_
    >
    > Alternatively, for patch releases, the folder structure is based on the patch release ID in a subfolder under the main release. For example, you can download the DHIS2 version 2.31.1 WAR release like this (replace `2.31` with your preferred version, and `2.31.1` with you\[r] preferred patch, if necessary):

    ```
    wget https://releases.dhis2.org/2.31/2.31.1/dhis.war
    ```

    > Move the WAR file into the Tomcat `webapps` directory. We want to call the WAR file `ROOT.war` in order to make it available at `localhost` directly without a context path:

    ```
    mv dhis.war tomcat-dhis/webapps/ROOT.war
    ```

    > DHIS2 should never be run as a privileged user. After you have modified the `setenv.sh` file, modify the startup script to check and see if the script has been invoked as `root`.

    ```
    #!/bin/sh
    set -e

    if [ "$(id -u)" -eq "0" ]; then
    echo "This script must NOT be run as root" 1>&2
    exit 1
    fi

    export CATALINA_BASE="/home/dhis/tomcat-dhis"
    /usr/share/tomcat7/bin/startup.sh
    echo "Tomcat started"
    ```

9. Running DHIS2

    > DHIS2 can now be started by invoking:

    ```
    sudo -u dhis tomcat-dhis/bin/startup.sh
    ```

    > _Warning_
    >
    > The DHIS2 server should never be run as `root` or other privileged user.

    > DHIS2 can be stopped by invoking:

    ```
    sudo -u dhis tomcat-dhis/bin/shutdown.sh
    ```

    > To monitor the behavior of Tomcat the log is the primary source of information. The log can be viewed with the following command:

    ```
    tail -f tomcat-dhis/logs/catalina.out
    ```

    > Assuming that the WAR file is called `ROOT.war`, you can now access your DHIS2 instance at the following URL:
    >
    > http://localhost:8080
