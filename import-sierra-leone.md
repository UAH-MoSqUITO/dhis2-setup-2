Commands for importing the sample data from [DHIS2 downloads](https://www.dhis2.org/downloads)

```shell
wget https://s3-eu-west-1.amazonaws.com/databases.dhis2.org/sierra-leone/2.31/dhis2-db-sierra-leone.sql.gz
sudo -u dhis /home/dhis/tomcat-dhis/bin/shutdown.sh
sudo -u postgres dropdb dhis2
sudo -u postgres createdb -O dhis dhis2
time ( zcat dhis2-db-sierra-leone.sql.gz | sudo -i -u postgres psql -v ON_ERROR_STOP=1 -d dhis2 2>&1 > import-sierra-leone.$(date +%s-%F).log )
tail $(ls -1 import-sierra-leone.*.log | tail -n1)
sudo -u dhis /home/dhis/tomcat-dhis/bin/startup.sh
```

_Log_

```console
vagrant@vagrant:~$ wget https://s3-eu-west-1.amazonaws.com/databases.dhis2.org/sierra-leone/2.31/dhis2-db-sierra-leone.sql.gz
--2019-03-20 21:34:13--  https://s3-eu-west-1.amazonaws.com/databases.dhis2.org/sierra-leone/2.31/dhis2-db-sierra-leone.sql.gz
Resolving s3-eu-west-1.amazonaws.com (s3-eu-west-1.amazonaws.com)... 52.218.105.202
Connecting to s3-eu-west-1.amazonaws.com (s3-eu-west-1.amazonaws.com)|52.218.105.202|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 80390342 (77M) [application/x-sql]
Saving to: ‘dhis2-db-sierra-leone.sql.gz’

dhis2-db-sierra-leone.sql. 100%[=====================================>]  76.67M  29.2MB/s    in 2.6s

2019-03-20 21:34:17 (29.2 MB/s) - ‘dhis2-db-sierra-leone.sql.gz’ saved [80390342/80390342]

vagrant@vagrant:~$ sudo -u dhis /home/dhis/tomcat-dhis/bin/shutdown.sh
Using CATALINA_BASE:   /home/dhis/tomcat-dhis
Using CATALINA_HOME:   /usr/share/tomcat8
Using CATALINA_TMPDIR: /home/dhis/tomcat-dhis/temp
Using JRE_HOME:        /usr/lib/jvm/java-8-openjdk-amd64/
Using CLASSPATH:       /usr/share/tomcat8/bin/bootstrap.jar:/usr/share/tomcat8/bin/tomcat-juli.jar
Mar 20, 2019 9:34:38 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/home/dhis/tomcat-dhis/common/classes], exists: [false], isDirectory: [false], canRead: [false]
Mar 20, 2019 9:34:38 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/home/dhis/tomcat-dhis/common], exists: [false], isDirectory: [false], canRead: [false]
Mar 20, 2019 9:34:38 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/usr/share/tomcat8/common/classes], exists: [false], isDirectory: [false], canRead: [false]
Mar 20, 2019 9:34:38 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/usr/share/tomcat8/common], exists: [false], isDirectory: [false], canRead: [false]
Mar 20, 2019 9:34:38 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/home/dhis/tomcat-dhis/server/classes], exists: [false], isDirectory: [false], canRead: [false]
Mar 20, 2019 9:34:38 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/home/dhis/tomcat-dhis/server], exists: [false], isDirectory: [false], canRead: [false]
Mar 20, 2019 9:34:38 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/usr/share/tomcat8/server/classes], exists: [false], isDirectory: [false], canRead: [false]
Mar 20, 2019 9:34:38 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/usr/share/tomcat8/server], exists: [false], isDirectory: [false], canRead: [false]
Mar 20, 2019 9:34:38 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/home/dhis/tomcat-dhis/shared/classes], exists: [false], isDirectory: [false], canRead: [false]
Mar 20, 2019 9:34:38 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/home/dhis/tomcat-dhis/shared], exists: [false], isDirectory: [false], canRead: [false]
Mar 20, 2019 9:34:38 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/usr/share/tomcat8/shared/classes], exists: [false], isDirectory: [false], canRead: [false]
Mar 20, 2019 9:34:38 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/usr/share/tomcat8/shared], exists: [false], isDirectory: [false], canRead: [false]
Tomcat stopped
vagrant@vagrant:~$ sudo -u postgres dropdb dhis2
vagrant@vagrant:~$ sudo -u postgres createdb -O dhis dhis2
vagrant@vagrant:~$ time ( zcat dhis2-db-sierra-leone.sql.gz | sudo -i -u postgres psql -v ON_ERROR_STOP=1 -d dhis2 2>&1 > import-sierra-leone.$(date +%s-%F).log )

real    4m20.015s
user    0m4.905s
sys     0m0.552s
vagrant@vagrant:~$ tail $(ls -1 import-sierra-leone.*.log | tail -n1)
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
vagrant@vagrant:~$ sudo -u dhis /home/dhis/tomcat-dhis/bin/startup.sh
Using CATALINA_BASE:   /home/dhis/tomcat-dhis
Using CATALINA_HOME:   /usr/share/tomcat8
Using CATALINA_TMPDIR: /home/dhis/tomcat-dhis/temp
Using JRE_HOME:        /usr/lib/jvm/java-8-openjdk-amd64/
Using CLASSPATH:       /usr/share/tomcat8/bin/bootstrap.jar:/usr/share/tomcat8/bin/tomcat-juli.jar
Tomcat started.
Tomcat started
```
