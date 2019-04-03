#ctap

This is the parent project of *ctap-web* and *ctap-feature*. It includes only the POM definition and some docs. The actual code of the CTAP projects is in the two modules. 

## Installation

If you want to install the CTAP web module on your own server, you can either use the readymade [dockerized version](https://gitlab.inf.unibz.it/commul/docker/ctap) or follow these steps.

1. Install git, maven, tomcat, postgres and jdk
```
apt-get update && apt-get install -y git maven tomcat8 openjdk-8-jdk postgresql
```
2. Get all the source code from github

For the CTAP that only works for English:

```
mkdir /opt/ctap
git clone https://github.com/ctapweb/ctap.git /opt/ctap
git clone https://github.com/ctapweb/ctap-web.git /opt/ctap/ctap-web
git clone https://github.com/ctapweb/ctap-feature.git /opt/ctap/ctap-feature
```

For the CTAP that works for English and German:
```
mkdir /opt/ctap
git clone https://github.com/ctapweb/ctap.git /opt/ctap
git clone -b ctap_german --depth=1 https://github.com/zweiss/multilingual-ctap-web.git /opt/ctap/ctap-web
git clone -b ctap_german --depth=1 https://github.com/zweiss/multilingual-ctap-feature.git /opt/ctap/ctap-feature
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
If you encounter the following error as one of us did:
```
--- maven-war-plugin:2.2:war (default-war) @ ctap-web ---
[WARNING] Error injecting: org.apache.maven.plugin.war.WarMojo
com.google.inject.ProvisionException: Unable to provision, see the following errors:

1) Error injecting constructor, java.lang.NoClassDefFoundError: com/thoughtworks/xstream/io/HierarchicalStreamDriver
  at org.apache.maven.plugin.war.WarMojo.<init>(WarMojo.java:49)
  while locating org.apache.maven.plugin.war.WarMojo

1 error
    at com.google.inject.internal.InjectorImpl$2.get(InjectorImpl.java:1025)
    ...
    at org.codehaus.plexus.classworlds.launcher.Launcher.main(Launcher.java:356)
Caused by: java.lang.NoClassDefFoundError: com/thoughtworks/xstream/io/HierarchicalStreamDriver
    at org.apache.maven.plugin.war.AbstractWarMojo.<init>(AbstractWarMojo.java:316)
    at org.apache.maven.plugin.war.WarMojo.<init>(WarMojo.java:49)
 	...
    at com.google.inject.internal.InjectorImpl$2.get(InjectorImpl.java:1012)
    ... 42 more
Caused by: java.lang.ClassNotFoundException: com.thoughtworks.xstream.io.HierarchicalStreamDriver
    at org.codehaus.plexus.classworlds.strategy.SelfFirstStrategy.loadClass(SelfFirstStrategy.java:50)
    at org.codehaus.plexus.classworlds.realm.ClassRealm.unsynchronizedLoadClass(ClassRealm.java:271)
    at org.codehaus.plexus.classworlds.realm.ClassRealm.loadClass(ClassRealm.java:247)
    at org.codehaus.plexus.classworlds.realm.ClassRealm.loadClass(ClassRealm.java:239)
    ... 58 more
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO]
[INFO] Common Text Analysis Platform ...................... SUCCESS [  0.213 s]
[INFO] Text Feature Extraction Module for CTAP ............ SUCCESS [ 16.827 s]
[INFO] The Web Module for CTAP ............................ FAILURE [01:29 min]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:47 min
[INFO] Finished at: 2019-04-01T17:51:13+02:00
[INFO] Final Memory: 47M/431M
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-war-plugin:2.2:war (default-war) on project ctap-web: Execution default-war of goal org.apache.maven.plugins:maven-war-plugin:2.2:war failed: Unable to load the mojo 'war' in the plugin 'org.apache.maven.plugins:maven-war-plugin:2.2'. A required class is missing: com/thoughtworks/xstream/io/HierarchicalStreamDriver
[ERROR] -----------------------------------------------------
[ERROR] realm =    plugin>org.apache.maven.plugins:maven-war-plugin:2.2
[ERROR] strategy = org.codehaus.plexus.classworlds.strategy.SelfFirstStrategy
[ERROR] urls[0] = file:/home/nadiushka/.m2/repository/org/apache/maven/plugins/maven-war-plugin/2.2/maven-war-plugin-2.2.jar
[ERROR] urls[1] = file:/home/nadiushka/.m2/repository/org/apache/maven/reporting/maven-reporting-api/2.0.6/maven-reporting-api-2.0.6.jar
[ERROR] urls[2] = file:/home/nadiushka/.m2/repository/org/apache/maven/doxia/doxia-sink-api/1.0-alpha-7/doxia-sink-api-1.0-alpha-7.jar
[ERROR] urls[3] = file:/home/nadiushka/.m2/repository/commons-cli/commons-cli/1.0/commons-cli-1.0.jar
[ERROR] urls[4] = file:/home/nadiushka/.m2/repository/org/codehaus/plexus/plexus-interactivity-api/1.0-alpha-4/plexus-interactivity-api-1.0-alpha-4.jar
[ERROR] urls[5] = file:/home/nadiushka/.m2/repository/org/apache/maven/maven-archiver/2.5/maven-archiver-2.5.jar
[ERROR] urls[6] = file:/home/nadiushka/.m2/repository/org/codehaus/plexus/plexus-io/2.0.2/plexus-io-2.0.2.jar
[ERROR] urls[7] = file:/home/nadiushka/.m2/repository/org/codehaus/plexus/plexus-archiver/2.1/plexus-archiver-2.1.jar
[ERROR] urls[8] = file:/home/nadiushka/.m2/repository/org/codehaus/plexus/plexus-interpolation/1.15/plexus-interpolation-1.15.jar
[ERROR] urls[9] = file:/home/nadiushka/.m2/repository/junit/junit/3.8.1/junit-3.8.1.jar
[ERROR] urls[10] = file:/home/nadiushka/.m2/repository/com/thoughtworks/xstream/xstream/1.3.1/xstream-1.3.1.jar
[ERROR] urls[11] = file:/home/nadiushka/.m2/repository/xpp3/xpp3_min/1.1.4c/xpp3_min-1.1.4c.jar
[ERROR] urls[12] = file:/home/nadiushka/.m2/repository/org/codehaus/plexus/plexus-utils/3.0/plexus-utils-3.0.jar
[ERROR] urls[13] = file:/home/nadiushka/.m2/repository/org/apache/maven/shared/maven-filtering/1.0-beta-2/maven-filtering-1.0-beta-2.jar
[ERROR] Number of foreign imports: 1
[ERROR] import: Entry[import  from realm ClassRealm[project>com.ctapweb:ctap-web:1.0.0-SNAPSHOT, parent: ClassRealm[maven.api, parent: null]]]
[ERROR]
[ERROR] -----------------------------------------------------: com.thoughtworks.xstream.io.HierarchicalStreamDriver
[ERROR] -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR]
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/PluginContainerException
[ERROR]
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :ctap-web
```
Delete maven, manually delete the .m2 folder with all its content, reinstall maven, reinstall the 2 .jars from point 3 and run mvn clean install.

Check the contents of the resulting .war archive by comparing it to the contents of the file war-contents.txt obtained with the command:

jar tvf ~/.m2/repository/com/ctapweb/ctap-web/1.0.0-SNAPSHOT/ctap-web-1.0.0-SNAPSHOT.war > war-contents.txt

This file lists the contents of a war that works for English and German.


8. Initialize the database by going to http://localhost:8080/ctapWebApp#initdb (the password is the one set as INITDBPASSWD in ctap-web/src/main/java/com/ctapweb/web/server/ServerProperties.java)

9.  Login with the email and password from ctap-web/src/main/java/com/ctapweb/web/server/ServerProperties.java and go to the Analysis Engine at the bottom of the left column (http://localhost:8080/ctap-web-1.0.0-SNAPSHOT/#adminae), push the button "Import AE"
