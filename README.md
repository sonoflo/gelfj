GELFJ - JBoss/Wildfly module
============================
see https://github.com/t0xa/gelfj for common usage (A GELF Appender for Log4j and a GELF Handler for JDK Logging)

Downloading
-----------

Add the following dependency section to your pom.xml:

    <dependencies>
      ...
      <dependency>
        <groupId>org.graylog2</groupId>
        <artifactId>gelfj</artifactId>
        <version>1.1.9</version>
        <scope>compile</scope>
      </dependency>
      ...
    </dependencies>

Or clone this repository, run a Maven packaging <code>mvn package</code> and get the jar !


What is the difference with t0xa GELFJ
--------------------------------------

I removed the stack trace from message field and add a new field named full_stack.
When using t0xa as a JBoss/Wildfly module, I had some issues with long stack traces : the whole log didn't show up on my graylog server.
So after a little debugging, the message was successfully sent but graylog didn't want these logs. I decided to make a separate field with the stack just to see if this would have fixed it and it worked ! I just want to share my work and help users who are stuck with this issue.


How to use GELFJ as a JBoss/Wildfly module
------------------------------------------
Steps from https://developer.jboss.org/thread/222855?tstart=0

First, add required modules to JBoss

- Get the jar as explained in the Downloading section
- copy the .jar file from gelfj/target/ directory to/as {jboss/wildfly}/modules/org/graylog2/logging/main/gelfj.jar (create directories as needed)
- create the following module.xml file in the same directory:

module.xml:

    <module xmlns="urn:jboss:module:1.1" name="org.graylog2.logging">    
        <resources>    
            <resource-root path="gelfj.jar"/>    
        </resources>    
        <dependencies>    
            <module name="json-simple"/>    
        </dependencies>    
    </module>   

- Download json-simple.jar from https://code.google.com/p/json-simple/downloads/list //Seeking for a github now that google code will be down soon, else I'll post the jar here later
- Copy the jar to/as {jboss/wildfly}/modules/json-simple/main/json-simple.jar
- Create the following module.xml file in the same directory:

module.xml:

    <module xmlns="urn:jboss:module:1.1" name="json-simple">    
        <resources>    
            <resource-root path="json-simple.jar"/>    
        </resources>    
        <dependencies>    
        </dependencies>    
    </module>

Second, customize your standalone configuration file
- Open your relevant "standalone.xml" file and go to \<subsystem xmlns="urn:jboss:domain:logging:x.x">
- Add the following custom-handler right under this "subsystem":

custom-handler:

    <custom-handler name="GRAYLOG2" class="org.graylog2.logging.GelfHandler" module="org.graylog2.logging">
        <level name="INFO"/> <!-- or what you want... --> 
        <properties>
            <property name="graylogHost" value="tcp:xxx.xxx.xxx.xxx"/> <!-- or udp: => add your graylog-server IP here -->
            <property name="graylogPort" value="tcp:xxx.xxx.xxx.xxx"/> <!-- optional, default 12201 -->
            <property name="extractStacktrace" value="true"/>
            <property name="facility" value="What you want"/> <!-- see graylog doc to see if this is useful in your case --> 
        </properties>
    </custom-handler>

- Add this handler to the list of handlers in root-logger:

root-logger:

    <root-logger>
        <level name="INFO"/>
        <handlers>
            <!-- other handlers here -->
            <handler name="GRAYLOG2"/>
        </handlers>
    </root-logger>

Third, restart JBoss and check startup logs. You shouldn't see any WARN or worse about Graylog/GelfHandler and logs should go directly to Graylog server.

Now you're good to go !


Options
-------

GelfAppender supports the following options:

- **graylogHost**: Graylog2 server where it will send the GELF messages; to use UDP instead of TCP, prefix with `udp:`
- **graylogPort**: Port on which the Graylog2 server is listening; default 12201 (*optional*)
- **originHost**: Name of the originating host; defaults to the local hostname (*optional*)
- **extractStacktrace** (true/false): Add stacktraces to the GELF message; default false (*optional*)
- **facility**: Facility which to use in the GELF message; default "gelf-java"
- **amqpURI**: AMQP URI (*required when using AMQP integration*)
- **amqpExchangeName**: AMQP Exchange name - should be the same as setup in graylog2-radio (*required when using AMQP integration*)
- **amqpRoutingKey**: AMQP Routing key - should be the same as setup in graylog2-radio (*required when using AMQP integration*)
- **amqpMaxRetries**: Retries count; default value 0 (*optional*)

What is GELF
------------

The Graylog Extended Log Format (GELF) avoids the shortcomings of classic plain syslog:

- Limited to length of 1024 byte
- Not much space for payloads like stacktraces
- Unstructured. You can only build a long message string and define priority, severity etc.

You can get more information here: [http://www.graylog2.org/about/gelf](http://www.graylog2.org/about/gelf)
