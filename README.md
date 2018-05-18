# AAI-ONAP-Setup
Date: 2018-04-26

Note: I tested it on 2018-04-26 and every thing is working fine. May be in next versions, ONAP team change file paths or names so consult instructions given on the following page.
        https://wiki.onap.org/pages/viewpage.action?pageId=25440974
-------------------------------------------------------------------------------

Ref Instructions:
https://wiki.onap.org/pages/viewpage.action?pageId=25440974

- Fresh Installation:
    - Environment: ubuntu 16.04.4 LTS with username : aaiadmin 

1. Create user:
    - $ sudo adduser aaiadmin
    - $ sudo usermod -aG sudo aaiadmin
    
    
Restart system and login as aaiadmin otherwise it will create problem during database creation phase

2. install openjdk 8

    $ sudo apt install openjdk-8-jdk

3. Install maven

    $ sudo apt-get install maven

4. Install git

    $ sudo apt-get install git

5. Install single node hadoop/titan

    $ wget http://s3.thinkaurelius.com/downloads/titan/titan-1.0.0-hadoop1.zip
    $ unzip titan-1.0.0-hadoop1.zip
    $ cd titan-1.0.0-hadoop1
    $ sudo ./bin/titan.sh start

If it starts successful it displays following messages on console:

	Forking Cassandra...
	Running `nodetool statusthrift`.. OK (returned exit status 0 and printed string "running").
	Forking Elasticsearch...
	Connecting to Elasticsearch (127.0.0.1:9300)... OK (connected to 127.0.0.1:9300).
	Forking Gremlin-Server...
	Connecting to Gremlin-Server (127.0.0.1:8182)... OK (connected to 127.0.0.1:8182).
	Run gremlin.sh to connect.


6. Install haproxy
    
    
    - $ sudo apt-get -y install haproxy
    - $ haproxy -v
    	HA-Proxy version 1.6.3 2015/12/25
	    Copyright 2000-2015 Willy Tarreau <willy@haproxy.org>

    Download and copy haproxy.cfg <wget https://wiki.onap.org/download/attachments/25440974/haproxy.cfg?version=1&modificationDate=1520955643000&api=v2> file in /etc/haproxy

    - $ sudo cp haproxy.cfg /etc/haproxy

    download aai.pem <wget https://wiki.onap.org/download/attachments/25440974/aai.pem?version=1&modificationDate=1520955643000&api=v2>
    
    - $ sudo cp aai.pem /etc/ssl/private/aai.pem
    
    - $ sudo chmod 640 /etc/ssl/private/aai.pem
    
    - $ sudo chown root:ssl-cert /etc/ssl/private/aai.pem
    
    - $ sudo mkdir /usr/local/etc/haproxy

    Add these host names to the loopback interface in /etc/hosts: 

        127.0.0.1 localhost aai-traversal.api.simpledemo.openecomp.org aai-resources.api.simpledemo.openecomp.org
    - $ sudo service haproxy restart

7. Set up repos. First, follow the initial setup instructions in Setting Up Your Development Environment <https://wiki.onap.org/display/DW/Setting+Up+Your+Development+Environment>


    - $ mkdir -p ~/LF/AAI
    - $ cd ~/LF/AAI
    - $ git clone ssh://<username>@gerrit.onap.org:29418/aai/aai-common
    - $ git clone ssh://<username>@gerrit.onap.org:29418/aai/traversal
    - $ git clone ssh://<username>@gerrit.onap.org:29418/aai/resources
    - $ git clone ssh://<username>@gerrit.onap.org:29418/aai/logging-service
    If you did not originally create a settings.xml file when setting up the dev environment, you may get an error on some of the repos saying that operant is unresolvable.  Using the example settings.xml <wget https://jira.onap.org/secure/attachment/10829/settings.xml>
    
     -$ cp settings.xml /home/aaiadmin/.m2

8. Build aai-common, traversal, and resources

    - $ cd ~/LF/AAI/aai-common
    - $ mvn -DskipTests clean install       #Should result in BUILD SUCCESS
    - $ cd ~/LF/AAI/resources
    - $ mvn -DskipTests clean install       #Should result in BUILD SUCCESS
    - $ cd ~/LF/AAI/logging-service
    - $ mvn -DskipTests clean install      #Should result in BUILD SUCCESS
    - $ cd ~/LF/AAI/traversal
    I had to add the following to traversal/pom.xml to get traversal to build: 

    <repositories>
                <repository>
                        <id>maven-restlet</id>
                        <name>Restlet repository</name>
                        <url>https://maven.restlet.com</url>
                </repository>
    </repositories>

    - $ mvn -DskipTests clean install      #Should result in BUILD SUCCESS

9. Titan setup

    Modify both titan-cached.properties and titan-realtime.properties to the following (for all MS’s that will connect to the local Cassandra backend)
    Chnage storage.backend=cassandra and storage.hostname=localhost properties in following files:
 
    - ~/LF/AAI/resources/aai-resources/src/main/resources/etc/appprops/janusgraph-cached.properties
    - ~/LF/AAI/resources/aai-resources/src/main/resources/etc/appprops/janusgraph-realtime.properties
    - ~/LF/AAI/traversal/aai-traversal/src/main/resources/etc/appprops/janusgraph-cached.properties
    - ~/LF/AAI/traversal/aai-traversal/src/main/resources/etc/appprops/janusgraph-realtime.properties
    
    The following property can be added to specify the keyspace name, each time you do this step (g) should be done. If not specified Titan will try to create/use a defaulted keyspace named titan.
    storage.cassandra.keyspace=<keyspace name>
    
10. Create datbase:
    - $ sudo mkdir -p /opt/app/aai-resources/lib
    - $ sudo cp ~/LF/AAI/resources/aai-resources/target/aai-resources-1.2.0-SNAPSHOT.jar /opt/app/aai-resources/lib
    - $ sudo cp -r /home/aaiadmin/LF/AAI/resources/aai-resources/src/main/resources /opt/app/aai-resources/
    - $ sudo cp ~/LF/AAI/resources/aai-resources/src/main/docker/aai.sh /etc/profile.d/aai.sh
    - $ sudo chmod 755 /etc/profile.d/aai.sh
    - $ cd ~/LF/AAI/resources/aai-resources/src/main/scripts
    - $ chmod +x createDBSchema.sh # If createDBSchema.sh is not recognized as command
    - $ ./createDBSchema.sh 
Note: may be it will complain  bash: ./createDBSchema.sh: /bin/ksh: bad interpreter: No such file or directory 
To solve this problem, you need to open createDBSchema.sh and replace /bin/ksh with /bin/bash

To check that database is successfully created run following command:

    - download apache-cassandra-3.11.2: http://cassandra.apache.org/download/
    - Extract it
    - iii) cd to the bin directory of apache-cassandra-x.xx.x 
    - run following command 
    - $./cqlsh localhost
    - cqlsh> describe tables;
        It will show all keyspaces and tables including Keyspace janusgraph

    From the resources MS run the create db schema standalone program.

    NOTE: The first thing that would need to be done is adding the schema to the local instance. (this will need to be done whenever using a new keyspace or after wiping the data).
    Here's the command I used, and it worked:
    - $ cd ~/LF/AAI/resources; java -DAJSC_HOME=aai-resources -DBUNDLECONFIG_DIR=src/main/resources/ -Dloader.main=org.onap.aai.dbgen.GenTester -jar aai-resources/target/aai-resources-1.2.0-SNAPSHOT.jar

11. Start the "resources" microservice
    Resources runs on port 8447.  Go to the resources directory
    - $ cd ~/LF/AAI/resources
    Set the debug port to 9447
    - $ export MAVEN_OPTS="-Xms1024m -Xmx5120m -XX:PermSize=2024m -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,address=9447,server=y,suspend=n"
    Start the microservice
    - $ java -DAJSC_HOME=aai-resources -DBUNDLECONFIG_DIR=src/main/resources/ -jar aai-resources/target/aai-resources-1.2.0-SNAPSHOT.jar 

12. Start the "traversal" microservice

    Traversal runs on port 8446.  Go to the traversal directory
    - $ cd ~/LF/AAI/traversal
    Set the debug port to 9446
    - $ export MAVEN_OPTS="-Xms1024m -Xmx5120m -XX:PermSize=2024m -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,address=9446,server=y,suspend=n"
    Start the microservice
    - $ java -DAJSC_HOME=aai-traversal -DBUNDLECONFIG_DIR=src/main/resources/ -jar aai-traversal/target/aai-traversal-1.2.0-SNAPSHOT.jar
    Should see something like this: 2017-07-26 12:46:35.524:INFO:oejs.Server:com.att.ajsc.runner.Runner.main(): Started @25827ms


13. To verify the resources and traversal microservice (this example uses Postman utility for Google Chrome), please visit : https://wiki.onap.org/pages/viewpage.action?pageId=25440974
    Note: If postmain starts complaing about SSL certificate verification then you need to close SSL certificate verification option. (Postmain --> Settings --> SSL Certification Verification) 
	
Good Luck !
