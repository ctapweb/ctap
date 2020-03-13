#ctap

This is the parent project of [ctap-web](https://github.com/commul/multilingual-ctap-web) (the GUI) and [ctap-feature](https://github.com/commul/multilingual-ctap-feature) (the feature extraction system). It includes only the POM definition and some docs. The actual code of the CTAP projects is in the two modules. 

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

For the CTAP that works for English, German and Italian:
```
mkdir /opt/ctap
git clone https://github.com/commul/ctap.git /opt/ctap
git clone -b ctap_german --depth=1 https://github.com/commul/multilingual-ctap-web.git /opt/ctap/ctap-web
git clone -b ctap_german --depth=1 https://github.com/commul/multilingual-ctap-feature.git /opt/ctap/ctap-feature
```

3. Install the two not publicly available dependencies locally
```
mkdir /opt/sources
cd /opt/sources
curl -L -o org.moxieapps.gwt.highcharts-1.7.0.jar https://downloads.sourceforge.net/project/gwt-highcharts/1.7.0/org.moxieapps.gwt.highcharts-1.7.0.jar
curl -L -o org.moxieapps.gwt.uploader-1.1.0.jar https://downloads.sourceforge.net/project/gwt-uploader/1.1.0/org.moxieapps.gwt.uploader-1.1.0.jar
curl -L -o anna-3.61.jar https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/mate-tools/anna-3.61.jar
mvn install:install-file -Dfile=/opt/sources/org.moxieapps.gwt.highcharts-1.7.0.jar -DgroupId=org.moxieapps.gwt -DartifactId=highcharts -Dversion=1.7.0 -Dpackaging=jar
mvn install:install-file -Dfile=/opt/sources/org.moxieapps.gwt.uploader-1.1.0.jar -DgroupId=org.moxieapps.gwt -DartifactId=uploader -Dversion=1.1.0 -Dpackaging=jar
mvn install:install-file -Dfile=/opt/sources/anna-3.61.jar -DgroupId=com.googlecode.mate-tools -DartifactId=anna -Dversion=3.61 -Dpackaging=jar
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

Check the contents of the resulting .war archive by comparing it to the contents of the file war-contents.txt obtained with the command:

jar tvf ~/.m2/repository/com/ctapweb/ctap-web/1.0.0-SNAPSHOT/ctap-web-1.0.0-SNAPSHOT.war > war-contents.txt

This file lists the contents of a war that works for English, German and Italian.


8. Initialize the database by going to http://localhost:8080/ctapWebApp#initdb (the password is the one set as INITDBPASSWD in ctap-web/src/main/java/com/ctapweb/web/server/ServerProperties.java)

9.  Go to the URL http://localhost:8080/ctap-web-1.0.0-SNAPSHOT/#signup and create an account with the email and password from ctap-web/src/main/java/com/ctapweb/web/server/ServerProperties.java. Once loged in, go to the Analysis Engine at the bottom of the left column (http://localhost:8080/ctap-web-1.0.0-SNAPSHOT/#adminae), push the button "Import AE". Wait a couple of seconds. A list of AEs will appear on the page.

10. Follow the multilingual.ctap-user-guide for instructions on how to use CTAP.

11. For developers, there is multilingual-CTAP-documentation.odt


```
@ARTICLE{chenmeurersctap,
       author = {{Chen}, X. and {Meurers}, D.},
        title = "{CTAP: A Web-Based Tool Supporting Automatic Complexity Analysis}",
      volume = {Proceedings of the Workshop on Computational Linguistics for Linguistic Complexity (CL4LC)},
     keywords = {linguistic complexity - readability - Italian - text analysis - cross-lingual analysis},
         year = 2016,
        month = December,
       adsurl = {https://www.aclweb.org/anthology/W16-4113}
}
```

```
@ARTICLE{okininafreyweissctap,
       author = {{Okinina}, N., {Frey}, J. C. and {Weiss}, Z.},
        title = "{CTAP for Italian: Integrating Components for the Analysis of Italian into a Multilingual Linguistic Complexity Analysis Tool}",
      volume = {Proceedings of the Twelfth International Conference on Language Resources and Evaluation (LREC 2020)},
     keywords = {linguistic complexity - readability - Italian - text analysis - cross-lingual analysis},
         year = 2020,
        month = May
}
```

