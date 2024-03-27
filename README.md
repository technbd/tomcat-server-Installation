
## Tomcat Server: 
To install Apache Tomcat on CentOS 7, you can follow these steps:


### Prerequisites:
- Java 


```
java -version
echo $JAVA_HOME
```

### Download Tomcat:
You can download Tomcat from the Apache Tomcat website using wget command this link "https://downloads.apache.org/tomcat/tomcat-9/". Make sure to replace the version number you want to ownload.
For example: 

```
wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.86/bin/apache-tomcat-9.0.86.zip
```


```
unzip apache-tomcat-9.0.86.zip
mv apache-tomcat-9.0.86 tomcat-9
```


### Start Tomcat:

Start the Tomcat by navigating to the bin directory inside the Tomcat directory and executing the startup.sh script:

```
cd /opt/tomcat-9/bin/
chmod +x *
```


```
./startup.sh
```

Verify that Tomcat has started successfully by accessing it in your web browser at **http://your_server_ip:8080**


## Advanced:
### Tomcat Web Management Interface:

At this point Tomcat is installed, and we can access it with a web browser on port 8080, but we can not access the web management interface because we have not created a user yet. Tomcat users and their roles are defined in the tomcat-users.xml file.


```
vim tomcat-9/conf/tomcat-users.xml


<?xml version="1.0" encoding="UTF-8"?>

<tomcat-users>

<!--
    Comments
-->

<!-- Add this line:  -->
   <role rolename="admin-gui"/>
   <role rolename="manager-gui"/>
   <role rolename="manager-script"/>
   <role rolename="manager-jmx"/>
   <role rolename="manager-status"/>
   <user username="admin" password="admin_password" roles="admin-gui,manager-gui,manager-script,manager-jmx,manager-status"/>
   <user username="developer" password="developer" roles="manager-script"/>


</tomcat-users>


save and quit
```


By default Tomcat web management interface is configured to allow access only from the **localhost**. If you want to be able to access the web interface from a remote IP or from anywhere which is not recommended because it is a security risk. 

To enable the Manager App for remote access, edit the following context.xml file:


```
vim tomcat-9/webapps/manager/META-INF/context.xml


<?xml version="1.0" encoding="UTF-8"?>


<Context antiResourceLocking="false" privileged="true" >

  <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                   sameSiteCookies="strict" />

<!-- comment this Valve directive section: -->
<!--
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->

  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>

</Context>


save and quit
```



To enable the Host Manager App for remote access, edit the following context.xml file:


```
vim tomcat-9/webapps/host-manager/META-INF/context.xml


<?xml version="1.0" encoding="UTF-8"?>

<Context antiResourceLocking="false" privileged="true" >
  <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                   sameSiteCookies="strict" />

<!-- comment this Valve directive section: -->
<!--
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->

  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>

</Context>


save and quit
```


```
./shutdown.sh
./startup.sh
```


### Test the Installation:

Tomcat web application manager dashboard is available at **http://your_server_ip:8080/manager/html**. From here, you can deploy, undeploy, start, stop, and reload your applications.


---
---


### Tomcat Default Port Change:
To change the default port of Apache Tomcat, typically port 8080, to another port (e.g., 8081), follow these steps:



```
vim tomcat-9/conf/server.xml


<?xml version="1.0" encoding="UTF-8"?>

<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <!-- Security listener. Documentation at /docs/config/listeners.html
  <Listener className="org.apache.catalina.security.SecurityListener" />
  
  ...
  ...

<Service name="Catalina">
  ...
  ...

  <!-- Port Change here:  -->
  <Connector port="8081" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               maxParameterCount="1000"
               />

  ...
  ...

  </Service>
</Server>


save and quit
```


```
./shutdown.sh
./startup.sh
```


```
netstat -tlpn | grep 8081
tcp6       0      0 :::8081     :::*       LISTEN      44444/java
```


Verify that Tomcat has started successfully by accessing it in your web browser at **http://your_server_ip:8081**



### For SSL (Self-signed):

Enabling SSL (Secure Socket Layer) for Apache Tomcat involves configuring Tomcat to use HTTPS instead of HTTP, which encrypts the communication between the server and clients. 

Tomcat currently operates only on JKS, PKCS11 or PKCS12 format keystores. The JKS format is Java's standard "Java KeyStore" format, and is the format created by the keytool command-line utility. This tool is included in the JDK. The PKCS12 format is an internet standard, and can be manipulated via (among other things) OpenSSL and Microsoft's Key-Manager.

You need an SSL certificate and key to enable SSL. You can either obtain a certificate from a trusted certificate authority (CA) or generate a self-signed certificate for testing purposes. Here's how to generate a self-signed certificate using Java's **keytool**:


```
ll $JAVA_HOME/bin/keytool
```


```
keytool -genkey -keyalg RSA -alias tomcat -keysize 2048 -validity 365  -keystore /opt/tomcat-9/test-ssl/self-signed-ssl.jks

    Enter keystore password: tomcat
    Re-enter new password:

    What is your first and last name?
  [Unknown]:  IDEA LAB
What is the name of your organizational unit?
  [Unknown]:  IT
What is the name of your organization?
  [Unknown]:  IDEA
What is the name of your City or Locality?
  [Unknown]:  Dhaka
What is the name of your State or Province?
  [Unknown]:  Dhaka
What is the two-letter country code for this unit?
  [Unknown]:  BD
Is CN=IDEA, OU=IT, O=IDEA, L=Dhaka, ST=Dhaka, C=BD correct?
  [no]:  yes

Enter key password for <tomcat>
        (RETURN if same as keystore password): [Hit Enter]


Warning:
The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore /opt/tomcat-9/test-ssl/self-signed-ssl.jks -destkeystore /opt/tomcat-9/test-ssl/self-signed-ssl.jks -deststoretype pkcs12".
```


```
### Verify:

keytool -list -v -keystore /opt/tomcat-9/test-ssl/self-signed-ssl.jks
    Enter keystore password:
```



**Tomcat can use three different implementations of SSL:**

- JSSE implementation provided as part of the Java runtime
- JSSE implementation that uses OpenSSL
- APR implementation, which uses the OpenSSL engine by default


```
vim tomcat-9/conf/server.xml


<?xml version="1.0" encoding="UTF-8"?>

<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <!-- Security listener. Documentation at /docs/config/listeners.html
  <Listener className="org.apache.catalina.security.SecurityListener" />
  
  ...
  ...

<Service name="Catalina">
  ...
  ...

  <!-- Port Change here:  -->
  <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               maxParameterCount="1000"
               />


  <!-- Add this for SSL:  -->
  <Connector port="8443" protocol="HTTP/1.1"
           maxThreads="150" maxParameterCount="1000"
           scheme="https" secure="true" SSLEnabled="true"
           keystoreFile="/opt/tomcat-9/test-ssl/self-signed-ssl.jks"
           keystorePass="tomcat"
           keyAlias="tomcat"
           clientAuth="false" sslProtocol="TLS" />

  ...
  ...

  </Service>
</Server>


save and quit
```



```
./shutdown.sh
./startup.sh
```


```
netstat -tlpn | grep 8443
tcp6       0      0 :::8443      :::*       LISTEN      85998/java
```



Verify that Tomcat has started successfully by accessing it in your web browser at **https://your_server_ip:8443**



### Redirect HTTP to HTTPS:
To force tomcat to redirect and revert all requested HTTP traffic to HTTPS, You need to edit "web.xml" configuration files. Scroll to the bottom of the file and add the following just above the </web-app> entry:


```
vim tomcat-9/conf/web.xml


<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
  version="4.0">

...
...

<!-- Force HTTPS, required for HTTP redirect! -->
<security-constraint>

 <web-resource-collection>
   <web-resource-name>Secured</web-resource-name>
   <url-pattern>/*</url-pattern>
 </web-resource-collection>

 <!-- auth-constraint goes here if you require authentication -->
 <user-data-constraint>
   <transport-guarantee>CONFIDENTIAL</transport-guarantee>
 </user-data-constraint>

</security-constraint>

</web-app>


save and quit
```


```
./shutdown.sh
./startup.sh
```


That's it! You've successfully installed Apache Tomcat on CentOS 7. Make sure to adjust the version numbers and paths according to your specific setup.


