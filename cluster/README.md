## Configuring Tomcat Cluster:
Clustering is two or more independent interconnected systems (vm or nodes) interlinked to provide reliability. Reliability can come in the form of higher availability (HA,) improved scalability, improved application availability and ease of maintenance.

Tomcat clustering is a group of Tomcat instances that are connected. An instance of Tomcat is an independent system. Clustering instances of Tomcat makes them interconnected.


### Pre-requisites:
- Java
- Two or more node
- Uniquely naming each node


### Cluster Topology:
- Node1: 192.168.10.190
- Node2: 192.168.10.191


### Host to IP address mapping:

```
vim /etc/hosts

192.168.10.190  node1
192.168.10.191  node2
```


### Tomcat Clustering some of Components:

Before going to session replication we need to understand 2 important concepts:
- Multicast or Static Membership
- Session Manager

**1. Multicast:** Choose a mechanism for cluster node discovery and membership. Multicast-based or static IP-based membership can be configured in the Cluster element. Multicast is To transmit a single message to a select group of recipients. here multicast used by tomcat cluster to identify the instances those part of cluster. 

There is 2 types of cluster:
- **Static Tomcat Cluster**: In static cluster there is no need multicast, because each tomcat we statically defined/configured the other instances. 

- **Dynamic Tomcat Cluster**: Dynamic Cluster we are not defined anything. so each tomcat in that cluster some how to identify the other tomcat instances.




**2. Session Manager:** is manages session data and ensures session replication across cluster nodes.. In Servlet Specification request.getSession(); line is mention that container (tomcat) is responsible for create the session. here tomcat use the Session Manager for this purpose.

There is 4 types of Session Manager:
- Standard Manager
- Persistent Manager
- Delta Manager
- Backup Manager


_Here are some of the important default values:_

- Multicast address is 228.0.0.4
- Multicast port is 45564 (the port and the address together determine cluster membership).
- The IP broadcasted is "**java.net.InetAddress.getLocalHost().getHostAddress()**" (make sure you don't broadcast 127.0.0.1, this is a common error)
- The TCP port listening for replication messages is the first available server socket in range 4000-4100
- Listener is configured "**ClusterSessionListener**"
- Two interceptors are configured "**TcpFailureDetector**" and "**MessageDispatchInterceptor**"



### Configure Tomcat Clustering (Node-1):

Before making the required changes let's take a backup of the default server.xml file. Find the following lines and change localhost to the IP address of each Tomcat Node. 

```
"<Engine name="Catalina" defaultHost="localhost">"
"<Host name="localhost"  appBase="webapps"

like,

"<Engine name="Catalina" defaultHost="192.168.10.190">"
"<Host name="192.168.10.190"  appBase="webapps"
```


According to Tomcat docs you need to do one of these two (NOT both):

- Specify in web.xml <distributable/> to enable clustering and use the default <Manager> specified in the server.xml or,

- Add a <Manager> at the application level inside context.xml file in "conf/context.xml"



```
vim tomcat-9/conf/server.xml

<Server port="8005" shutdown="SHUTDOWN">

  ...
  ...

  <Service name="Catalina">
    ...
    ...

    <Engine name="Catalina" defaultHost="192.168.10.190">
    ...
    ...

    <!--
      <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
    -->

    <!-- Add for this cluster -->
    <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"
                         channelSendOptions="8">

                <!--
                <Manager className="org.apache.catalina.ha.session.BackupManager"
                   expireSessionsOnShutdown="false"
                   notifyListenersOnReplication="true"
                   mapSendOptions="6"/>
                -->

                <Manager className="org.apache.catalina.ha.session.DeltaManager"
                            expireSessionsOnShutdown="false"
                            notifyListenersOnReplication="true"/>

          <Channel className="org.apache.catalina.tribes.group.GroupChannel">
            <Membership className="org.apache.catalina.tribes.membership.McastService"
                                            address="228.0.0.4"
                                            port="45564"
                                            frequency="500"
                                            dropTime="3000"/>

            <Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
                                      address="auto"
                                      port="4000"
                                      autoBind="100"
                                      selectorTimeout="5000"
                                      maxThreads="6"/>

            <Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">
                <Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>
            </Sender>

            <Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatchInterceptor"/>
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.ThroughputInterceptor"/>

        </Channel>

        <!--
        <Valve className="org.apache.catalina.ha.tcp.ReplicationValve"
                 filter=".*\.gif|.*\.js|.*\.jpeg|.*\.jpg|.*\.png|.*\.htm|.*\.html|.*\.css|.*\.txt"/>
        -->

        <Valve className="org.apache.catalina.ha.tcp.ReplicationValve" filter=""/>
        <Valve className="org.apache.catalina.ha.session.JvmRouteBinderValve"/>

        <!--
        <Deployer className="org.apache.catalina.ha.deploy.FarmWarDeployer"
                    tempDir="/tmp/war-temp/"
                    deployDir="/tmp/war-deploy/"
                    watchDir="/tmp/war-listen/"
                    watchEnabled="false"/>
        -->

        <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>

     </Cluster>
     
     ...
     ...

      <Host name="192.168.10.190"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

      ...
      ...

      </Host>
    </Engine>

  </Service>
</Server>


save and quit
```


```
./configtest.sh
./shutdown.sh
./startup.sh
```


```
netstat -tlpn

tcp6    0    0 127.0.0.1:8005          :::*     LISTEN    82384/java
tcp6    0    0 :::8080                 :::*     LISTEN    82384/java
tcp6    0    0 192.168.10.190:4000     :::*     LISTEN    82384/java
```


```
tail -f tomcat-9/logs/catalina.out
```



### Configure Tomcat Clustering (Node-2):

```
vim tomcat-9/conf/server.xml

<Server port="8005" shutdown="SHUTDOWN">

  ...
  ...

  <Service name="Catalina">
    ...
    ...

    <Engine name="Catalina" defaultHost="192.168.10.191">
    ...
    ...

    <!--
      <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
    -->

    <!-- Add for this cluster -->
    <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"
                         channelSendOptions="8">

                <!--
                <Manager className="org.apache.catalina.ha.session.BackupManager"
                   expireSessionsOnShutdown="false"
                   notifyListenersOnReplication="true"
                   mapSendOptions="6"/>
                -->

                <Manager className="org.apache.catalina.ha.session.DeltaManager"
                            expireSessionsOnShutdown="false"
                            notifyListenersOnReplication="true"/>

          <Channel className="org.apache.catalina.tribes.group.GroupChannel">
            <Membership className="org.apache.catalina.tribes.membership.McastService"
                                            address="228.0.0.4"
                                            port="45564"
                                            frequency="500"
                                            dropTime="3000"/>

            <Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
                                      address="auto"
                                      port="4000"
                                      autoBind="100"
                                      selectorTimeout="5000"
                                      maxThreads="6"/>

            <Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">
                <Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>
            </Sender>

            <Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatchInterceptor"/>
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.ThroughputInterceptor"/>

        </Channel>

        <!--
        <Valve className="org.apache.catalina.ha.tcp.ReplicationValve"
                 filter=".*\.gif|.*\.js|.*\.jpeg|.*\.jpg|.*\.png|.*\.htm|.*\.html|.*\.css|.*\.txt"/>
        -->

        <Valve className="org.apache.catalina.ha.tcp.ReplicationValve" filter=""/>
        <Valve className="org.apache.catalina.ha.session.JvmRouteBinderValve"/>

        <!--
        <Deployer className="org.apache.catalina.ha.deploy.FarmWarDeployer"
                    tempDir="/tmp/war-temp/"
                    deployDir="/tmp/war-deploy/"
                    watchDir="/tmp/war-listen/"
                    watchEnabled="false"/>
        -->

        <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>

     </Cluster>
     
     ...
     ...

      <Host name="192.168.10.191"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

      ...
      ...

      </Host>
    </Engine>

  </Service>
</Server>


save and quit
```


```
./configtest.sh
./shutdown.sh
./startup.sh
```


```
netstat -tlpn

tcp6    0    0 127.0.0.1:8005          :::*     LISTEN    2296/java
tcp6    0    0 :::8080                 :::*     LISTEN    2296/java
tcp6    0    0 192.168.10.191:4000     :::*     LISTEN    2296/java
```


```
tail -f tomcat-9/logs/catalina.out
```




### Make application distributable:
In the web.xml of all web applications (node1 and node2) desired to have failover, this "<distributable />" tag must be added within <web-app> and </web-app> tags:


```
vim tomcat-9/webapps/ROOT/WEB-INF/web.xml


<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
  version="4.0"
  metadata-complete="true">

  <!-- Add this line -->
  <distributable />

  <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to Tomcat
  </description>

</web-app>


save and quit
```


### Links:
- [Tomcat Cluster](https://tomcat.apache.org/tomcat-9.0-doc/cluster-howto.html)


Remember that this is a basic outline, and the actual configuration may vary depending on your specific requirements and environment.




