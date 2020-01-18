# JIRA plugin for http header authentication (SSO / Kerberos)

This plugin provides authentication based on a http header  (default: X_Forwarded_User). 
The authenticator will fall back to the default JIRA authenticator, so everything external should keep working as expected. 
WARNING: This plugin is currently not actively developed or maintained. 
It was created for an organisation that no longer does or uses FOSS, so it was moved. 
Feel free to use it at your own risk, or to fork and improve it. 
I hope it is helpful anyway.

## License 

This software is distributed under the MIT License. See COPYING for details. 


## Build
In order to build the package, there must be downloaded and registered in maven repo some libraries
which are not available in the maven repo due to license restrictions from a well known company for such stuff.

see:  https://examples.javacodegeeks.com/software-development/atlassian/setting-system-atlassian-jira-add-development/

Download the following libs:
https://www.oracle.com/java/technologies/java-archive-downloads-java-plat-downloads.html#7110-jndi-1.2.1-oth-JPR
https://download.oracle.com/otndocs/jcp/7089-jta-1.0.1B-mr-class-oth-JSpec/?submit=Download

rename them to jar

Install to your local Maven Repo:
mvn install:install-file -DgroupId=jndi -DartifactId=jndi -Dversion=1.2.1 -Dpackaging=jar -Dfile=$HOME/Downloads/jndi-1_2_1.jar
mvn install:install-file -DgroupId=jta -DartifactId=jta -Dversion=1.0.1 -Dpackaging=jar -Dfile=$HOME/Downloads/jta-1_0_1B-classes.jar


Next is to register the Atlassian Maven repos in your $HOME/.m2/settings.xml. Extend it with the following repository profiles:

			<repositories>
				<repository>
					<id>atlassian-public</id>
					<url>https://maven.atlassian.com/repository/public</url>
					<snapshots>
						<enabled>true</enabled>
						<updatePolicy>never</updatePolicy>
						<checksumPolicy>warn</checksumPolicy>
					</snapshots>
					<releases>
						<enabled>true</enabled>
						<checksumPolicy>warn</checksumPolicy>
					</releases>
				</repository>
				<repository>
					<id>atlassian-plugin-sdk</id>
					<url>file://${HOME}/.m2/repository</url>
					<snapshots>
						<enabled>true</enabled>
					</snapshots>
					<releases>
						<enabled>true</enabled>
						<checksumPolicy>warn</checksumPolicy>
					</releases>
				</repository>
			</repositories>
			<pluginRepositories>
				<pluginRepository>
					<id>atlassian-public</id>
					<url>https://maven.atlassian.com/repository/public</url>
					<releases>
						<enabled>true</enabled>
						<checksumPolicy>warn</checksumPolicy>
					</releases>
					<snapshots>
						<updatePolicy>never</updatePolicy>
						<checksumPolicy>warn</checksumPolicy>
					</snapshots>
				</pluginRepository>
				<pluginRepository>
					<id>atlassian-plugin-sdk</id>
					<url>file://${HOME}/.m2/repository</url>
					<releases>
						<enabled>true</enabled>
						<checksumPolicy>warn</checksumPolicy>
					</releases>
					<snapshots>
						<enabled>true</enabled>
					</snapshots>
				</pluginRepository>
			</pluginRepositories>



Build it with mvn install


Alternative way to build
* Get the Atlassian SDK as described at [Atlassian](https://developer.atlassian.com/display/DOCS/Set+up+the+Atlassian+Plugin+SDK+and+Build+a+Project)
* Build the .jar file with the `atlas-package` command in the root folder (containing the pom.xml) 



## Install in Jira
There are a few things you need in order to install this plugin:

* Stop your JIRA instance if it is running
* Copy the target/russo-1.0.jar file to the WEB-INF/libs folder of your JIRA installation
* Modify the WEB-INF/classes/seraph-config.xml file by commenting out existing auth classes and adding ```<authenticator class="ch.fuchsnet.seraph.RussoAuthenticator"/>```
* Restart your JIRA instance
* If it doesn't work as expected, check your JIRA logs. If you need more verbose information, set useDebug to true and recompile and reinstall the package

## Configuring your httpd

In order to get it to work, you need to configure your httpd (e.g. Apache httpd) to do the authentication and set the header. 
For security reasons you should make sure that user-set headers are removed, otherwise users will be able to spoof authentication
and log in as a different user! 

Example Apache configuration

```
<VirtualHost *:443>
SSLEngine on
SSLCertificateFile /etc/pki/tls/certs/mypubliccert.pem
SSLCertificateKeyFile /etc/pki/tls/private/privatekey.pem
ProxyPreserveHost On
ProxyRequests Off
ServerName jira.mycompany.tld
ProxyPass / http://localhost:8080/
ProxyPassReverse / http://localhost:8080/
SSLProxyEngine On

    <Location />
        AuthType Kerberos
        AuthName "Jira Kerberos Auth"
        KrbMethodNegotiate On
        KrbMethodK5Passwd On
        KrbAuthRealms MYREALM
        Krb5KeyTab /etc/httpd/httpd.keytab
        KrbLocalUserMapping On
        require valid-user
        RequestHeader set X-Forwarded-User %{REMOTE_USER}s
    </Location>
</VirtualHost>

```
