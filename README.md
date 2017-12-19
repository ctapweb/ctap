#ctap

This is the parent project of *ctap-web* and *ctap-feature*. It includes only the POM definition and some docs. The actual code of the CTAP projects is in the two moduals. 

## Installation

If you want to install the CTAP web module on your own server, you can either use the readymade [dockerized version](https://gitlab.inf.unibz.it/commul/docker/ctap) or follow these steps.

1. Install git, maven, tomcat, postgres and jdk
```
apt-get update && apt-get install -y git maven tomcat8 openjdk-8-jdk postgresql
```
2. Get all the source code from github
```
mkdir /opt/ctap
git clone https://github.com/ctapweb/ctap.git /opt/ctap
git clone https://github.com/ctapweb/ctap-web.git /opt/ctap/ctap-web
git clone https://github.com/ctapweb/ctap-feature.git /opt/ctap/ctap-feature
```
3. Install the two not publicly available dependencies locally
```
mkdir /opt/sources
cd /opt/sources
curl -L -o org.moxieapps.gwt.highcharts-1.7.0.jar https://downloads.sourceforge.net/project/gwt-highcharts/1.7.0/org.moxieapps.gwt.highcharts-1.7.0.jar
curl -L -o org.moxieapps.gwt.uploader-1.1.0.jar https://downloads.sourceforge.net/project/gwt-uploader/1.1.0/org.moxieapps.gwt.uploader-1.1.0.jar
mvn install:install-file -Dfile=/opt/sources/org.moxieapps.gwt.highcharts-1.7.0.jar -DgroupId=org.moxieapps.gwt -DartifactId=highcharts -Dversion=1.7.0 -Dpackaging=jar
mvn install:install-file -Dfile=/opt/sources/org.moxieapps.gwt.uploader-1.1.0.jar -DgroupId=org.moxieapps.gwt -DartifactId=uploader -Dversion=1.1.0 -Dpackaging=jar
```
4. Edit  `/opt/ctap/ctap-web/src/main/java/com/ctapweb/web/server/ServerProperties.java` and set the DBPASSWD and the INITDBPASSWD.
5. Create the user and the database in postgres
```
/etc/init.d/postgresql start
su postgres
psql
postgres=# create user ctap with password 'ctap';
postgres=# create database ctap with owner ctap;
postgres=# grant all privileges on database ctap to ctap;
exit
```
6. Edit `pg_hba.conf` (e.g. at `/etc/postgresql/9.6/main/pg_hba.conf`) to make sure that the authentication method md5 is allowed
7. Install the app and start tomcat
```
mvn clean install
cp ~/.m2/repository/com/ctapweb/ctap-web/1.0.0-SNAPSHOT/ctap-web-1.0.0-SNAPSHOT.war /usr/local/tomcat/webapps/ctapWebApp.war
/etc/init.d/tomcat8 start
```
8. Initialize the database by going to http://localhost:8080/ctapWebApp#initdb (the password is the one set as INITDBPASSWD in ServerProperties.java)
