# Using Oracle Maven Repository

Do you need to include Oracle JDBC jars in your Maven project?
It is not as simple as with other Maven repositories, because you need to set up authentication.
If you have not done that before, it might be quite a challenge.
This guide should help you get it done.
So let's get started.

First of all, include the dependency in your pom.xml (adjust _artifactId_ and _version_ as you need)
```
<dependency>
	<groupId>com.oracle.jdbc</groupId>
	<artifactId>ojdbc8</artifactId>
	<version>12.2.0.1</version>
</dependency>
```
Additionally, the Oracle maven repositories need to be added to your _pom.xml_
```
<repositories>
	<repository>
		<id>maven.oracle.com</id>
		<releases>
			<enabled>true</enabled>
		</releases>
		<snapshots>
			<enabled>false</enabled>
		</snapshots>
		<url>https://maven.oracle.com</url>
		<layout>default</layout>
	</repository>
</repositories>
<pluginRepositories>
	<pluginRepository>
		<id>maven.oracle.com</id>
		<url>https://maven.oracle.com</url>
	</pluginRepository>
</pluginRepositories>
```
If you go to https://maven.oracle.com, you will see, that ...
- Directory browsing is not allowed on the Oracle Maven Repository
- Registration is required to access the Oracle Maven Repository. To register, please visit the [registration site](http://www.oracle.com/webapps/maven/register/license.html).
- Technical information about how to use the Oracle Maven Repository is available [here](https://maven.oracle.com/doc.html).

Following the technical information, you need to use a master password. 
Encrypt it using
```
$ mvn --encrypt-master-password masterPassw0rd
{3dH9rX5YDZgH9bArLT8icLidlqjwjcdcy0iGuXigBB8=}
```
and save it in _settings-security.xml_, which is located in your $HOME/.m2 folder (the same folder where your _settings.xml_ file is stored)
```
$ cat $HOME/.m2/settings-security.xml
<settingsSecurity>
  <master>
    {3dH9rX5YDZgH9bArLT8icLidlqjwjcdcy0iGuXigBB8=}
  </master>
</settingsSecurity>
```
Next, you need to encrypt your [Oracle Technology Network (OTN)](http://www.oracle.com/technetwork/index.html) password.
If you have no OTN account yet, you need to register [here](https://profile.oracle.com/myprofile/account/create-account.jspx).
```
$ mvn --encrypt-password yourOTNpassw0rd
{ifRwrvBQqZAH0ipAAmoLrGWAIk5HLQznilXWNnnAdus=}
```
Finally, add server for Oracle Maven repository to your _$HOME/.m2/settings.xml_ file
```
<servers>
  <server>
    <id>maven.oracle.com</id>
    <username>your.name@example.com</username>
    <password>{ifRwrvBQqZAH0ipAAmoLrGWAIk5HLQznilXWNnnAdus=}</password>
    <configuration>
      <basicAuthScope>
        <host>ANY</host>
        <port>ANY</port>
        <realm>OAM 11g</realm>
      </basicAuthScope>
      <httpConfiguration>
        <all>
          <params>
            <property>
              <name>http.protocol.allow-circular-redirects</name>
              <value>%b,true</value>
            </property>
          </params>
        </all>
      </httpConfiguration>
    </configuration>
  </server>
</servers>
```
And you're done. Your Maven project can now use any artifacts from Oracle Maven repository.
```
$ mvn install
...
```
### References
https://blogs.oracle.com/dev2dev/get-oracle-jdbc-drivers-and-ucp-from-oracle-maven-repository-without-ides

