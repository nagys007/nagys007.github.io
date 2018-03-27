Is your "java" command working
```
$ java -version

java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
```
but JAVA_HOME is not set?
```
$ echo $JAVA_HOME

$
```
There are some simple ways how to find out, where your JRE/JDK is located.
Like `which`
```
$ which java

/c/ProgramData/Oracle/Java/javapath/java
```
or `alternatives`
```
$ alternatives java
```
but what if they do not work? E.g. `which` might return `/usr/bin/java`, which is a link to one of the `alternatives`
```
$ which java
/usr/bin/java

$ ls -la /usr/bin/java
lrwxrwxrwx. 1 root root 22 Mar 27 12:43 /usr/bin/java -> /etc/alternatives/java
```

The question is: is there a generic way how to set your JAVA_HOME based on java command in your PATH. 
This can be interesting when scripting your infrastructure as code, for instance.

Let me show
```
$ java -XshowSettings:properties -version
Property settings:
    awt.toolkit = sun.awt.windows.WToolkit
    file.encoding = Cp1252
...
    java.home = C:\Program Files\Java\jre1.8.0_161
...
```
As *showSettings* uses stderr for output, we need to redirect and grep for line containing _java.home_
```
java -XshowSettings:properties -version 2>&1 | grep java.home
```
Now we just need to extract the value after the equal sign. Either using _awk_
```
export JAVA_HOME=$(java -XshowSettings:properties -version 2>&1 | grep java.home | awk -F"=" '{print $NF}')
```
or, if no _awk_ available in your system, then using _cut_
```
export JAVA_HOME=$(java -XshowSettings:properties -version 2>&1 | grep java.home | cut -d= -f2)
```
This should be now the result
```
$ echo $JAVA_HOME
C:\Program Files\Java\jre1.8.0_161
```
Enjoy!

### Appendix #1
Just noticed that the proposed solution does not work for NiFi
```
$ bin/nifi.sh start
nifi.sh: JAVA_HOME is not valid:  /usr/java/jdk1.8.0_162/jre
```
Obviously, NiFi requires JAVA_HOME without _jre_ folder
```
$ export JAVA_HOME=/usr/java/jdk1.8.0_162/
$ echo $JAVA_HOME
/usr/java/jdk1.8.0_162/
$ bin/nifi.sh status

Java home: /usr/java/jdk1.8.0_162/
NiFi home: /home/vagrant/nifi-1.5.0

Bootstrap Config File: /home/vagrant/nifi-1.5.0/conf/bootstrap.conf

2018-03-27 13:28:07,403 INFO [main] org.apache.nifi.bootstrap.Command Apache NiFi is currently running, listening to Bootstrap on port 51373, PID=3377
```
So how to get arouns this in a generic way?
```
export JAVA_HOME=$(java -XshowSettings:properties -version 2>&1 | grep java.home | \
  cut -d= -f2 | xargs -I % sh -c 'if [ "$(basename %)" == "jre" ]; then dirname %; else echo %; fi')
```
Fairly clumsy, I admit, but it works. Should you have a more elegant solution, please let me know.
