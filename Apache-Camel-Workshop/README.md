# IDS Plugfest 2021 08 Apache Camel Workshop

Hi there! 
This is a simple workshop with some labs to solve. 
The scope starts with simple Apache Camel routes dealing with "hello world" message, conditionanls and basic content modification.
The scope ends with the usage of Apache Camel Processors.

Each Lab contains sample key material for its own. 
- LabX\Start\etc contains the key material and truststore as well as some configuration
- LabX\Start\deploy contains a policy for a usage control engine called LUCON which just state, to let all data through
You do not need to worry about these things for now.

Requirements: Docker and docker-compose installed on a linux system

## Lab 1: Sending your first "hello world"

Switch to the folder Lab1_Basics. 
Here you can see a "Start" folder which has a code base which must be patched and a "Solution" folder which contains the finished adaptations, so far.
Within the Start folder switch to "example-getting-started". 
You can launch the first lab using 
```
cd Lab1_Basics\Start\example-getting-started
docker-compose up
```
Now you see the Trusted Connector launching and finally the file in same directory called "example-idscp2-localloop.xml" will be processed and its Camel Routes launched automatically. Hit Ctrl + C to stop the execution and perform a restart, which is recommended for any adaptation to the xml file.

The first task is simple and deals with the goal to understand the main structure of Camel Routes. 
Please modify the Camel Route in such a way that the string "Hello-world!" is sent to the receiving end of the connector instead of the message containing a timestamp. 
If you already have an idea what you should modify please go ahead and try. However, the precise steps are mentioned in the next lists.

1. Open the XML-file Lab1_Basics\Start\example-getting-started\example-idscp2-localloop.xml
2. Find the comment "Routes" on line 31. Here a CamelContext object can be seen in XML style which contains multiple <route>-elements. 
3. Find the route element with id="sendTime" and carefully observe the contents of this route.
4. The first <from>-node inside of this route uses a timer and launches this route every 10,000 milliseconds (10seconds). At this point a new empty message object within Apache Camel is generated and sequentially processed further. The next action that occurs is the <setBody>-node which gives the Camel message object contents.
5. You need to modify this statement to <simple>Hello-World!</simple> before the message object reaches the <to>-node which sends the message object out to the receiving end.
6. Now the receiver, the route with the id "receiveTime", checks whether the content is as requested and prints either a success-message to the console log or an error message.

Note: There is no need to use quotation marks inside of <simple> elements.

Apache Camel Routes allow you to use different if-then-else-structures to direct the message flow and its processing. It is also possible to perform many other comparison statements instead of just the equality check. You can observe this concept in the receiving route which shows a <choice> node followed by <when> node followed by a condition. A complete list can be seen in the following sources.

Additional source for <Simple>-node text processing: https://camel.apache.org/components/latest/languages/simple-language.html#_simple_language_options
Additional source for <choice>-node variable checking: https://camel.apache.org/components/latest/languages/simple-language.html#_operator_support

## Lab 2: Configuring Key Material for IDS-DAPS in Camel Route

In the previously lab you have learned about the Camel Route itself but in our sample XML files the following parts were an element of the file, also:
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:camel="http://camel.apache.org/schema/spring"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
          http://camel.apache.org/schema/spring
          http://camel.apache.org/schema/spring/camel-spring.xsd">
```
This part is required for any XML based Camel Route and contains general information about the structure and stylesheet of the XML file.

```
    <!-- Define TLS configuration for data consumer -->
    <camel:sslContextParameters id="serverSslContext">
        <camel:keyManagers keyPassword="password">
            <camel:keyStore resource="etc/consumer-keystore.p12" password="password"/>
        </camel:keyManagers>
        <camel:trustManagers>
            <camel:keyStore resource="etc/truststore.p12" password="password"/>
        </camel:trustManagers>
    </camel:sslContextParameters>

    <!-- Define TLS configuration for data provider -->
    <camel:sslContextParameters id="clientSslContext">
        <camel:keyManagers keyPassword="password">
            <camel:keyStore resource="etc/provider-keystore.p12" password="password"/>
        </camel:keyManagers>
        <camel:trustManagers>
            <camel:keyStore resource="etc/truststore.p12" password="password"/>
        </camel:trustManagers>
    </camel:sslContextParameters>
```
This part is a configuration which states which key material should be used for which route. 
For the routes "sendTime" and "receiveTime" which send information via the Apache Camel Adapter "IDSCP2" this is especially important.
The following elements must be configured and typically cause some trouble for people new to the IDS:
- <camel:keyStore> 
  - resource: is the path where the p12-archive is located (delievered after applying for it and needed to connect to the IDS-DAPS)
  - password: is the protection for the whole archive and NOT the additional layer of protection for a key stored inside of the archive. For PoC or test environments the default password "password" is used.
- <camel:keyManagers> is the element dealing with the p12 archive provided by the DAPS.
- <camel:trustManagers> is the element that contains a keystore with valid certificates to authenticate the IDS infrastructure (public keys only)

For this lab there is an issue. The SSL connection will not start up and the reason is: A wrong keystore is configured. 
Can you fix it by doing the following steps?

1. Inside the folder "Lab2_Setting_key_materials\etc" you can see that there is an additional p12 file called "localhost.p12" which is the target for configuration
2. Check out the file "docker-compose.yml" in "Lab2_Setting_key_materials\example-getting-started" and you can see that the file is already mapped to the inside of the docker container. Line 15 states that the hostpath is tranlated to /root/etc/localhost and /root/ is the current working directory for the application.
3. Please note that the default key material is issued for Domain names, such as consumer-core or provider-core. This is typical naming scheme for testing environments within docker-compose scripts. In reality there would complete unique domain names, such as google.de etc. But since the provider-keystore.p12 and the consumer-keystore.p12 are not fitting to the alias stated in the docker-compose file on line 24 with "localhost" no SSL connection can be established anymore.
4. In the XML-route file "example-idscp2-localloop.xml" change : <camel:keyStore resource="etc/consumer-core.p12" password="password"/> to <camel:keyStore resource="etc/localhost.p12" password="password"/>. The same for provider-core.
5. In the <route>-element in the following <to>-element use the new domain idscp2client://localhost:9292

After saving this and relaunching the connector using docker-compose up the system should be running smoothly again!



## Lab 3: File Access via Apache Camel

Apache Camel has over 200 different socalled Camel Adapters. These are the endpoints and startings point that are mentioned for the <from>- and the <to>-elements. So far you have seen the adapters timer, idscp2client and idscp2server. The next basic adapter that might be useful in practice is the file-adapter which allows writing to or reading from files.

Note: Normally each Camel-Adapter's code package must be included in the application. For instance for connectors using Maven as package manager, the pom.xml must include the corresponding Camel-Adapters artifact. 
For each available Camel-Adapter, the package which must be included is defined, but feel free to have a look for yourself:

1. Modify docker-compose.yml file by adding the volume      - ./outputdir:/root/outputdir (here we want to dump all message in file form)
2. Create folder outputdir inside the "docker-getting-started"-folder
3. Add the line <to uri="file://outputdir"/> in the receiving route, after <log message="Received via IDS protocol: ${body}"/>
4. You can already launch the "docker-compose up"-command but optionally you can add the parameter to the route also right after the file://outputdir statement "?fileName=resourcedata.txt" to specify a filename for the data. Notive the command line should print a slightly different success-message.

More info of all the numerous capabilities of Apache Camel can be found here: 
https://camel.apache.org/components/latest/languages/file-language.html#_samples



## Lab 4: HTTP-Requests via Apache Camel

Normally for this lab you have to include the following Apache Camel Adapters, but the docker-compose script uses containers which have these already installed.
- Install camel-http (https://camel.apache.org/components/latest/http-component.html)
- Install camel-netty-http (https://camel.apache.org/components/latest/netty-http-component.html)

You can see which adapters are running when you take a look at the console output during startup. You should see the following lines during startup.

When starting the connector it must state:
```
2021-08-24 11:13:04.487 DEBUG 67624 --- [           main] de.fhg.aisec.ids.TrustedConnector        : Component: camel-http
2021-08-24 11:13:04.487 DEBUG 67624 --- [           main] de.fhg.aisec.ids.TrustedConnector        : Component: camel-rest-api
2021-08-24 11:13:04.487 DEBUG 67624 --- [           main] de.fhg.aisec.ids.TrustedConnector        : Component: camel-rest
```

The aim for this lab is to send http-requests from Camel Routes to other adapters. 
1. Switch to the directory Lab4_Web_access\Start\example-getting-started
2. Take a look at the "example-idscp2-localloop.xml". It is basically working the same way as in the previous Camel Route examples.
3. Each Apache Camel Route can only process one, single <from>-statement but multiple <to>-statements. To achieve the next goal you can either add the following code below the already existing <to>-statement
```
    <route id="sendTime">
        <from uri="timer://tempPerSecond?fixedRate=true&amp;period=10000"/>
        <setBody>
            <simple>Message at $simple{date:now:yyyy-MM-dd HH:mm:ss}</simple>
        </setBody>
        <log message="Sending message body &quot;${body}&quot;..."/>
        <to uri="idscp2client://localhost:9292/?sslContextParameters=#clientSslContext" />
  
        <!-- Body must be set to null otherwise request won't work. -->
        <setBody>
          <simple>${null}</simple>
        </setBody>	
        <setHeader name="CamelHttpMethod">
          <constant>GET</constant>
        </setHeader>
        <log message="Sending the following http request to google.com: ${body}"/>
        <to uri="http://www.google.com"/>
        <log message="Http request was sent. Google responded: ${body}"/>
    </route>
```

4. As alternative an additional route could be added also:

```
      <route id="sendTime">
        <from uri="timer://tempPerSecond?fixedRate=true&amp;period=10000"/>
        <!-- no need to set the body, it must be empty anyways in this variant -->
        <setHeader name="CamelHttpMethod">
          <constant>GET</constant>
        </setHeader>
        <log message="Sending the following http request to google.com: ${body}"/>
        <to uri="http://www.google.com"/>
        <log message="Http request was sent. Google responded: ${body}"/>
      </route>
```

You can observe that the HTTP verb is provided by setting the header accordingly, the default value is a GET-statement.
Also, you can observer, that the reply of the server is immediately stored inside of the Camel message object, which is why the google.com's website code will be printed out to the console output after the successful request.

Camel adapters to receive requests can also be launched using, for instance, the netty-http Camel Adapter. Which is documented here: 
https://camel.apache.org/components/latest/netty-http-component.html

