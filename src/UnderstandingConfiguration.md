
# Understanding Configuration

This chapter describes the options to configure your Hazelcast applications and explains the utilities which you can make use of while configuring. You can configure Hazelcast using one or mix of the following options: 

* Declarative way
* Programmatic way
* Using Hazelcast system properties
* Within the Spring context

## Configuring Declaratively


This is the configuration option where you use an XML configuration file. When you download and unzip `hazelcast-<version>.zip`, you will see the following files present in `/bin` folder, which are standard XML-formatted configuration files:

* `hazelcast.xml`: Default declarative configuration file for Hazelcast. The configuration in this XML file should be fine for most of the Hazelcast users. If not, you can tailor this XML file according to your needs by adding/removing/modifying properties.
* `hazelcast-full-example.xml`: Configuration file which includes all Hazelcast configuration elements and attributes with their descriptions. It is the "superset" of `hazelcast.xml`. You can use `hazelcast-full-example.xml` as a reference document to learn about any element or attribute, or you can change its name to `hazelcast.xml` and start to use it as your Hazelcast configuration file.

A part of `hazelcast.xml` is shown as an example below.

```xml
<group>
	<name>dev</name>
	<password>dev-pass</password>
</group>
<management-center enabled="false">http://localhost:8080/mancenter</management-center>
<network>
	<port auto-increment="true" port-count="100">5701</port>
	<outbound-ports>
		<!--
		Allowed port range when connecting to other members.
		0 or * means the port provided by the system.
		-->
		<ports>0</ports>
	</outbound-ports>
	<join>
		<multicast enabled="true">
		<multicast-group>224.2.2.3</multicast-group>
		<multicast-port>54327</multicast-port>
		</multicast>
		<tcp-ip enabled="false">
```

### Composing Declarative Configuration

You can compose the declarative configuration of your Hazelcast member or Hazelcast client from multiple declarative configuration snippets. In order to compose a declarative configuration, you can use the `<import/>` element to load different declarative configuration files.

Let's say you want to compose the declarative configuration for Hazelcast out of two configurations: `development-group-config.xml` and `development-network-config.xml`. These two configurations are shown below.

`development-group-config.xml`:

```xml
<hazelcast>
  <group>
      <name>dev</name>
      <password>dev-pass</password>
  </group>
</hazelcast>
```
<br></br>

`development-network-config.xml`:

```xml
<hazelcast>
  <network>
    <port auto-increment="true" port-count="100">5701</port>
    <join>
        <multicast enabled="true">
            <multicast-group>224.2.2.3</multicast-group>
            <multicast-port>54327</multicast-port>
        </multicast>
    </join>
  </network>
</hazelcast>
```

To get your example Hazelcast declarative configuration out of the above two, use the `<import/>` element as shown below.


```xml
<hazelcast>
  <import resource="development-group-config.xml"/>
  <import resource="development-network-config.xml"/>
</hazelcast>
```

This feature also applies to the declarative configuration of Hazelcast client. Please see the following examples.


`client-group-config.xml`:

```xml
<hazelcast-client>
  <group>
      <name>dev</name>
      <password>dev-pass</password>
  </group>
</hazelcast-client>
```
<br></br>

`client-network-config.xml`:

```xml
<hazelcast-client>
    <network>
        <cluster-members>
            <address>127.0.0.1:7000</address>
        </cluster-members>
    </network>
</hazelcast-client>
```

To get a Hazelcast client declarative configuration from the above two examples, use the `<import/>` element as shown below.

```xml
<hazelcast-client>
  <import resource="client-group-config.xml"/>
  <import resource="client-network-config.xml"/>
</hazelcast>
```


<br></br>
![image](images/NoteSmall.jpg) ***NOTE:*** *Use `<import/>` element on top level of the XML hierarchy.*
<br></br>

Using the element `<import>`, you can also load XML resources from classpath and file system:

```xml
<hazelcast>
  <import resource="file:///etc/hazelcast/development-group-config.xml"/> <!-- loaded from filesystem -->
  <import resource="classpath:development-network-config.xml"/>  <!-- loaded from classpath -->
</hazelcast>
```

The element `<import>` supports placeholders too. Please see the following example snippet:

```xml
<hazelcast>
  <import resource="${environment}-group-config.xml"/>
  <import resource="${environment}-network-config.xml"/>
</hazelcast>
```



## Configuring Programmatically

Besides declarative configuration, you can configure your cluster programmatically. For this you can create a `Config` object, set/change its properties and attributes, and use this `Config` object to create a new Hazelcast member. Following is an example code which configures some network and Hazelcast Map properties.

```java
Config config = new Config();
config.getNetworkConfig().setPort( 5900 )
					.setPortAutoIncrement( false );
                
MapConfig mapConfig = new MapConfig();
mapConfig.setName( "testMap" )
					.setBackupCount( 2 );
					.setTimeToLiveSeconds( 300 );
        
config.addMapConfig( mapConfig );
```

To create a Hazelcast member with the above example configuration, pass the configuration object as shown below:

```
HazelcastInstance hazelcast = Hazelcast.newHazelcastInstance( config );
```

<br>
![image](images/NoteSmall.jpg) ***NOTE:*** *The `Config` must not be modified after the Hazelcast instance Hazelcast
has been started. In other words, all configuration must be completed before creating the `HazelcastInstance`*
<br>

You can also create a named Hazelcast member. In this case, you should set `instanceName` of `Config` object as shown below:

```java
Config config = new Config();
config.setInstanceName( "my-instance" );
Hazelcast.newHazelcastInstance( config );
```

To retrieve an existing Hazelcast member by its name, use the following:
    
```
Hazelcast.getHazelcastInstanceByName( "my-instance" );
```

To retrieve all existing Hazelcast members, use the following:

```
Hazelcast.getAllHazelcastInstances();
```

<br>
![image](images/NoteSmall.jpg) ***NOTE:*** *Hazelcast performs schema validation through the file `hazelcast-config-<version>.xsd` which comes with your Hazelcast libraries. Hazelcast throws a meaningful exception if there is an error in the declarative or programmatic configuration.*

<br>

If you want to specify your own configuration file to create `Config`, Hazelcast supports several ways including filesystem, classpath, InputStream, and URL:

- `Config cfg = new XmlConfigBuilder(xmlFileName).build();`
- `Config cfg = new XmlConfigBuilder(inputStream).build();`
- `Config cfg = new ClasspathXmlConfig(xmlFileName);`
- `Config cfg = new FileSystemXmlConfig(configFilename);`
- `Config cfg = new UrlXmlConfig(url);`
- `Config cfg = new InMemoryXmlConfig(xml);`


## Configuring with System Properties

You can use system properties to configure some aspects of Hazelcast. You set these properties as name and value pairs through declarative configuration, programmatic configuration or JVM system property. Following are examples for each option.

**Declaratively:**

```xml
  ....
  <properties>
    <property name="hazelcast.property.foo">value</property>
    ....
  </properties>
</hazelcast>
```

**Programmatically:**

```java
Config config = new Config() ;
config.setProperty( "hazelcast.property.foo", "value" );
```

**Using JVM's `System` class or `-D` argument:**

`System.setProperty( "hazelcast.property.foo", "value" );`

or

`java -Dhazelcast.property.foo=value`

You will see Hazelcast system properties mentioned throughout this Reference Manual as required in some of the chapters and sections. All Hazelcast system properties are listed in the [System Properties appendix](#system-properties) with their descriptions, default values and property types as a reference for you.



## Configuring within Spring Context

If you use Hazelcast with [Spring](https://spring.io/) you can declare beans using the namespace `hazelcast`. When you add the namespace declaration to the element `beans` in the Spring context file, you can start to use the namespace shortcut `hz` to be used as a bean declaration. Following is an example Hazelcast configuration when integrated with Spring:

```
<hz:hazelcast id="instance">
  <hz:config>
    <hz:group name="dev" password="password"/>
    <hz:network port="5701" port-auto-increment="false">
      <hz:join>
        <hz:multicast enabled="false"/>
        <hz:tcp-ip enabled="true">
          <hz:members>10.10.1.2, 10.10.1.3</hz:members>
        </hz:tcp-ip>
      </hz:join>
    </hz:network>
  </hz:config>
</hz:hazelcast>
```

Please see the [Spring Integration section](#spring-integration) for more information on Hazelcast-Spring integration.


## Checking Configuration

When you start a Hazelcast member without passing a `Config` object, as explained in the [Configuring Programmatically section](#configuring-programmatically), Hazelcast checks the member's configuration as follows:

-	First, it looks for the `hazelcast.config` system property. If it is set, its value is used as the path. This is useful if you want to be able to change your Hazelcast configuration; you can do this because it is not embedded within the application. You can set the `config` option with the following command:
 
	`- Dhazelcast.config=`*`<path to the hazelcast.xml>`*.
	
	The path can be a regular one or a classpath reference with the prefix `classpath:`.
-	If the above system property is not set, Hazelcast then checks whether there is a `hazelcast.xml` file in the working directory.
-	If not, it then checks whether `hazelcast.xml` exists on the classpath.
-	If none of the above works, Hazelcast loads the default configuration (`hazelcast.xml`) that comes with your Hazelcast package.

Before configuring Hazelcast, please try to work with the default configuration to see if it works for you. This default configuration should be fine for most of the users. If not, you can consider to modify the configuration to be more suitable for your environment.



## Using Wildcards

Hazelcast supports wildcard configuration for all distributed data structures that can be configured using `Config`, that is, for all except `IAtomicLong`, `IAtomicReference`. Using an asterisk (\*) character in the name, different instances of maps, queues, topics, semaphores, etc. can be configured by a single configuration.

A single asterisk (\*) can be placed anywhere inside the configuration name.

For instance, a map named `com.hazelcast.test.mymap` can be configured using one of the following configurations.

```xml
<map name="com.hazelcast.test.*">
...
</map>
```
```xml
<map name="com.hazel*">
...
</map>
```
```xml
<map name="*.test.mymap">
...
</map>
```
```xml
<map name="com.*test.mymap">
...
</map>
```

Or a queue '`com.hazelcast.test.myqueue`':

```xml
<queue name="*hazelcast.test.myqueue">
...
</queue>
```
```xml
<queue name="com.hazelcast.*.myqueue">
...
</queue>
```


## Using Variables

In your Hazelcast and/or Hazelcast Client declarative configuration, you can use variables to set the values of the elements. This is valid when you set a system property programmatically or you use the command line interface. You can use a variable in the declarative configuration to access the values of the system properties you set.

For example, see the following command that sets two system properties.

```
-Dgroup.name=dev -Dgroup.password=somepassword
```

Let's get the values of these system properties in the declarative configuration of Hazelcast, as shown below.

```xml
<hazelcast>
  <group>
    <name>${group.name}</name>
    <password>${group.password}</password>
  </group>
</hazelcast>
```

This also applies to the declarative configuration of Hazelcast Client, as shown below.

```xml
<hazelcast-client>
  <group>
    <name>${group.name}</name>
    <password>${group.password}</password>
  </group>
</hazelcast-client>
```

If you do not want to rely on the system properties, you can use the `XmlConfigBuilder` and explicitly set a `Properties` instance, as shown below.
 
```java
Properties properties = new Properties();

// fill the properties, e.g. from database/LDAP, etc.

XmlConfigBuilder builder = new XmlConfigBuilder();
builder.setProperties(properties)
Config config = builder.build();
HazelcastInstance hz = Hazelcast.newHazelcastInstance(config);
```
