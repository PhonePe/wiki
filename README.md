# [Primer](https://github.com/phaneesh/primer)

Primer is a low latency high throughput [JWT](https://jwt.io) service which can be used to secure API interactions. 
Primer is built with dropwizard and uses hystrix for fault tolerance & isolation.
Primer is build on Java 8 & compiles only on Java 8
 
## Dependencies
* dropwizard
* Hystrix
* Aerospike
* [mock-aerospike](https://github.com/srini156/mock-aersopike) 
## Usage
This service can be used as the token store which can be used to verify and validate the claims by api client.
More about JWT [here](https://jwt.io/)   

### Build instructions
  - Clone the source:

        git clone github.com/phaneesh/primer

  - Build

        mvn package
  - Run
        docker-compose up

## API Documentation
The service has swagger documentation can can be used to try and check api definitions at <host>:8080/swagger

# [Dropwizard Primer Bundle](https://github.com/phaneesh/primer-bundle)

This bundle adds Primer JWT service support for dropwizard.
This bundle compiles only on Java 8.
 
## Dependencies
* [Primer](https://github.com/phaneesh/primer)

## Usage
The bundle adds Primer JWT service support for dropwizard. 
This makes it easier to secure the your API with JWT and a robust claims negotiation.
Tokens can be either static (no expiry) or dynamic with expiry. 
As a added bonus; it supports declarative role based authorizations
 
### Build instructions
  - Clone the source:

        git clone github.com/phaneesh/primer-bundle

  - Build

        mvn install

### Maven Dependency
Use the following repository:
```xml
<repository>
    <id>clojars</id>
    <name>Clojars repository</name>
    <url>https://clojars.org/repo</url>
</repository>
```
Use the following maven dependency:
```xml
<dependency>
    <groupId>io.dropwizard.primer</groupId>
    <artifactId>primer-bundle</artifactId>
    <version>1.3.13-1</version>
</dependency>
```

### Using Primer bundle

#### Bootstrap
```java
    @Override
    public void initialize(final Bootstrap...) {
        bootstrap.addBundle(new PrimerBundle() {
            
            public PrimerBundleConfiguration getPrimerConfiguration() {
                ...
            }
            
            public Set<String> withWhiteList(T configuration) {
                ...
            }
            
            public PrimerAuthorizationMatrix withAuthorization(T configuration) {
                ...
            }
        });
    }
```

### Configuration
```yaml
primer:
  enabled: true
  authTypesEnabled:
    CONFIG: true
    ANNOTATION: true
  absentTokenStatus: BAD_REQUEST
  endpoint:
    type: simple
    host: my.primer.somewhere
    port: 8080
  cacheExpiry: 600
  cacheMaxSize: 100000
  clockSkew: 60
  prefix: Bearer
  privateKey: thisismynotsosecretkey 
  whileListUrl:
    - unprotected/url
    - unprotected/url/{with}/{path}/{param}
  authorizations:
    - type: dynamic #can be static, dynamic or auto (uses token to infer the type of auth)
      methods:
        - GET
      roles:
        - user
      url: protected/user/{do}/{some}/{stuff}
    - type: static
      methods:
        - POST
      roles:
        - admin
      url: protected/admin/{do}/{admin}/{stuff}  
```

# [Revolver](https://github.com/phaneesh/revolver)

This bundle enables one to build downstream proxy with support for transparent callbacks and durable mailbox with polling support.
This bundle compiles only on Java 8.

## Features
* Honours Accept header for common media types (JSON, XML, MsgPack)
* Polling for status of requests
* Durability of requests/responses
* Pluggable persistence provider for requests/responses
 
## Dependencies
* [dropwizard-xml](https://github.com/phaneesh/xml-bundle)
* [dropwizard-msgpack](https://github.com/phaneesh/msgpack-bundle)

## Usage
Build a downstream proxy with ease that enables transparent callback support 
 
### Build instructions
  - Clone the source:

        git clone github.com/phaneesh/revolver

  - Build

        mvn install

### Maven Dependency
Use the following repository:
```xml
<repository>
    <id>clojars</id>
    <name>Clojars repository</name>
    <url>https://clojars.org/repo</url>
</repository>
```
Use the following maven dependency:
```xml
<dependency>
    <groupId>io.dropwizard.revolver</groupId>
    <artifactId>dropwizard-revolver</artifactId>
    <version>1.3.7-18-SNAPSHOT</version>
</dependency>
```

### Using Revolver bundle

#### Bootstrap

```java
    @Override
    public void initialize(Bootstrap<ApiConfiguration> bootstrap) {
        bootstrap.addBundle(new RevolverBundle<ApiConfiguration>() {

            @Override
            public RevolverConfig getRevolverConfig(ApiConfiguration apiConfiguration) {
                return apiConfiguration.getRevolver();
            }

            @Override
            public PersistenceProvider getPersistenceProvider() {
                return new InMemoryPersistenceProvider();
            }
        });
```

#### Configuration
```yaml
revolver:
  clientConfig:
    clientName: revolver-api
  services:
    - type: http
      service: mocky
      connectionPoolSize: 5
      connectionKeepAliveInMillis: 60000
      authEnabled: false
      endpoint:
        type: simple
        host: www.mocky.io
        port: 80
      threadPoolGroupConfig:
        threadPools:
          - threadPoolName: group1-thread-pool
            concurrency: 5
            maxRequestQueueSize: 10
          - threadPoolName: group2-thread-pool
            concurrency: 5
            maxRequestQueueSize: 10
      apis:
        - api: ping
          async: false
          path: "{version}/56da42e80f0000ac31a427ce"
          whitelist: true #Optional metadata for external authentication & authorization systems. Omitting the config will not effect behaviour.
          methods:
            - GET
          authorization:  #Optional metadata for external authorization systems. Omitting the config will not effect behaviour  
            type: dynamic #can 
            methods:
                - GET
            roles:
                - user
          runtime:
            threadPool:
              concurrency: 5
              timeout: 10000
        - api: pong
          async: false
          path: "{version}/56da42e80f0000ac31a427ce"
          whitelist: true #Optional metadata for external authentication & authorization systems. Omitting the config will not effect behaviour.
          methods:
            - GET
          authorization:  #Optional metadata for external authorization systems. Omitting the config will not effect behaviour  
            type: dynamic #can 
            methods:
                - GET
            roles:
                - user
          runtime:
            threadPool:
              timeout: 10000
          groupThreadPool:
            enabled: true
            name: group1-thread-pool
        - api: retryer
          async: false
          path: "{version}/retry"
          whitelist: true #Optional metadata for external authentication & authorization systems. Omitting the config will not effect behaviour.
          methods:
            - GET
          authorization:  #Optional metadata for external authorization systems. Omitting the config will not effect behaviour  
            type: dynamic #can 
            methods:
                - GET
            roles:
                - user
          runtime:
            threadPool:
              timeout: 10000
          retryConfig:
            enabled: true
            maxRetry: 3
```

#### Dashboard
![Dashboard](images/dashboard.png)

# [Ranger](https://github.com/phaneesh/ranger)

Ranger is a high level service discovery framework built on Zookeeper. The framework brings the following to the table:
  - Support of sharding of the service provider nodes
  - Support for monitoring of service provider nodes
  - Provides type-safe generic interface for integration with support for custom serializers and deserializers
  - Provides simple ways to plug in custom shard and node selection
  - Fault tolerant client side discovery with a combination of watchers and polling on watched nodes

## Why?

As request rates increase, load balancers, even the very expensive ones, become bottlenecks. We needed to move beyond and be able to talk to services without having to channel all traffic through a load-balancer. There is obviously curator discovery; but as much as we love curator, we needed more features on top of it. As such we built this library to handle app level sharding and healtchecks. Btw, it still uses curator for low level ZK interactions.

## Usage
Ranger provides two types of discovery out of the box:
  - Simple unsharded service discovery with service provider node healthchecks
  - Sharded service discovery with service provider node healthchecks
We'll take these up, one by one.

###Build instructions
  - Clone the source:

        git clone github.com/flipkart-incubator/ranger

  - Build

        mvn install

### Maven Dependency
Use the following repository:
```
<repository>
    <id>clojars</id>
    <name>Clojars repository</name>
    <url>https://clojars.org/repo</url>
</repository>
```
Use the following maven dependency:
```
<dependency>
    <groupId>com.flipkart.ranger</groupId>
    <artifactId>ranger</artifactId>
    <version>0.4.1</version>
</dependency>
```
## How it works
There are service providers and service clients. We will look at the interactions from both sides.
### Service Provider
Service providers register to the Ranger system by building and starting a ServiceProvider instance. During registering itself the provider must provide the following:
- _ShardInfo type_ - This is a type parameter to the ServiceProvider class. This can be any class that can be serialized and deserialized by the Serializer and Deserializer provided (see below). The hashCode() for this class is used to match and find the matching shard for a query. So be sure to implement this properly. A special version of this class _UnshardedClusterInfo_ is provided for unsharded discovery.
- _Zookeeper details_ - Can be any one of the following:
  - _Connection String_ - Zookeeper connections string to connect to ZK cluster.
  - CuratorFramework object - A prebuilt CuratorFramework object.
- _Namespace_ - A namespace for the service. For example the team name this service belongs to.
- _Service Name_ - Name of the service to be used by client for discovery.
- _Host_ - Hostname for the service.
- _Port_ - Port on which this service is running.
- _Serializer_ - A serializer implementation that will be used to serialize and store the shard information on ZooKeeper
- _Healthcheck_ - The healthcheck function is called every second and the status is updated on Zookeeper. 
- _Monitors_ : The health state of your service is also decided by a list of monitors. Register a list of monitors. These monitors will be monitored at regular intervals and an aggregated status is updated on Zookeeper
  - _Isolated Monitors_ - Each of these monitors will be running continuously on separate isolated threads. Each thread holds an independent state of the isolated monitor. The state of all Monitors will be aggregated an updated on Zookeeper at regular intervals. 

A node will be marked unhealthy iff:
  - The service is stopped.
  - If any isolated monitor's state is _HealthcheckStatus.unhealthy_
  - If _Healthcheck.check()_ has not been updated for over a minute. This signifies that the process is probably zombified.

#### Registering a simple unsharded service
This is very simple. Use the following boilerplate code.
```
ServiceProvider<UnshardedClusterInfo> serviceProvider
            = ServiceProviderBuilders.unshardedServiceProviderBuilder()
                .withConnectionString("localhost:2181")           //Zookeeper host string
                .withNamespace("test")                            //Service namespace
                .withServiceName("test-service")                  //Service name
                .withSerializer(new Serializer<UnshardedClusterInfo>() { //Serializer for info
                    @Override
                    public byte[] serialize(ServiceNode<UnshardedClusterInfo> data) {
                        try {
                            return objectMapper.writeValueAsBytes(data);
                        } catch (JsonProcessingException e) {
                            e.printStackTrace();
                        }
                        return null;
                    }
                })
                .withHostname(host)                            //Service hostname
                .withPort(port)                                //Service port
                .withHealthcheck(new Healthcheck() {           //Healthcheck implementation.
                    @Override
                    public HealthcheckStatus check() {
                        return HealthcheckStatus.healthy;      // OOR stuff should be put here
                    }
                })
                .withIsolatedHealthMonitor(new RotationStatusMonitor(TimeEntity.everySecond(), "/var/rotation.html"))
                .buildServiceDiscovery();
        serviceProvider.start();                               //Start the instance
```
Stop the provider once you are done. (Generally this is when process ends)
```
serviceProvider.stop()
```
#### Registering a sharded service
Let's assume that the following is your shard info class:

```
   private static final class TestShardInfo {
        private int shardId;

        public TestShardInfo(int shardId) {
            this.shardId = shardId;
        }

        public TestShardInfo() {
        }

        public int getShardId() {
            return shardId;
        }

        public void setShardId(int shardId) {
            this.shardId = shardId;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;

            TestShardInfo that = (TestShardInfo) o;

            if (shardId != that.shardId) return false;

            return true;
        }

        @Override
        public int hashCode() {
            return shardId;
        }
    }
```

To register a service provider node with this shard info, we can use the following code:

```
final ServiceProvider<TestShardInfo> serviceProvider
    = ServiceProviderBuilders.<TestShardInfo>shardedServiceProviderBuilder()
                .withConnectionString("localhost:2181")
                .withNamespace("test")
                .withServiceName("test-service")
                .withSerializer(new Serializer<TestShardInfo>() {
                    @Override
                    public byte[] serialize(ServiceNode<TestShardInfo> data) {
                        try {
                            return objectMapper.writeValueAsBytes(data);
                        } catch (JsonProcessingException e) {
                            e.printStackTrace();
                        }
                        return null;
                    }
                })
                .withHostname(host)
                .withPort(port)
                .withNodeData(new TestShardInfo(shardId)) //Set the shard info for this shard
                .withHealthcheck(new Healthcheck() {
                    @Override
                    public HealthcheckStatus check() {
                        return HealthcheckStatus.healthy;
                    }
                })
                .buildServiceDiscovery();
        serviceProvider.start();
```

Stop the provider once you are done. (Generally this is when process ends)

```
serviceProvider.stop()
```

### Monitors
In a distributed architecture, taking care of thousands of servers is a difficult task. Failures are bound to happen, and individual services could always face issues. It becomes very important that we automate handling such failures. Ranger allows you to do that, for your _ServiceProviders_.

As mentioned earlier, the health state of any _ServiceProvider_ is determined by a set of health monitors which are continuously running in the Service Provider.
All monitors (and at least 1) need to be registered while building the _ServiceProvider_.

You may register any kind of _Monitor_, which could be monitoring any serivce/system level metric. For example, you could have monitors:
- that monitor the service's heap, to ensure that it doesn't go beyond a threashold
- that check for any breach in max jetty threads 
- that monitors the systems disk space
- that does a continuous ping test
- that monitors the 5XX count from the service.

If any of the above are breached, the service will automatically be marked as unhealthy.

- _Isolated Monitors_ - Any extention of _IsolatedHealthMonitor_ may be used to register an isolated monitor. Each of these monitors will be running continuously on separate isolated threads. Each thread holds an independent state of the isolated monitor. The state of all Monitors will be aggregated an updated on Zookeeper at regular intervals.
  - _PingCheckMonitor_ - This monitor can be used to ping a url at regular intervals. It could be a self localhost ping too. You can also add minimum failure counts, to ensure that there are no fluctuations
    ```
        .withIsolatedHealthMonitor(new PingCheckMonitor(new TimeEntity(2, TimeUnit.SECONDS), httpRequest, 5000, 5, 3, "google.com", 80));  // put in the url here
    ```
  - _RotationStatusMonitor_ - This monitor can be used check the rotation status of your server, which decides if the host can serve traffic at the moment or not. Removing the file, will automatically prevent this host from getting discovered.
    ```
        ..withIsolatedHealthMonitor(new RotationStatusMonitor(TimeEntity.everySecond(), "/var/rotation.html"));  // path of file to be checked 
    ```
At regular intervals, all of the above monitors will be aggregated into a single Health state of the service, which


### Service discovery
For service discovery, a _ServiceFinder_ object needs to be built and used.
- _ShardInfo type_ - This is a type parameter to the ServiceFinder class. This can be any class that can be serialized and deserialized by the Serializer and Deserializer provided (see below). The hashCode() for this class is used to match and find the matching shard for a query. So be sure to implement this properly. A special version of this class _UnshardedClusterInfo_ is provided for unsharded discovery.
- _Zookeeper details_ - Can be any one of the following:
  - _Connection String_ - Zookeeper connections string to connect to ZK cluster.
  - CuratorFramework object - A prebuilt CuratorFramework object.
- _Namespace_ - A namespace for the service. For example the team name this service belongs to.
- _Service Name_ - Name of the service to be used by client for discovery.
- _Host_ - Hostname for the service.
- _Port_ - Port on which this service is running.
- _Deserializer_ - A deserializer implementation that will be used to deserialize and select shard from zookeeper.

Depending on whether you are looking to access a sharded service or an unsharded service, the code will differ a little.

#### Finding an instance of an unsharded Service Provider
First build and start the finder.

```
UnshardedClusterFinder serviceFinder
            = ServiceFinderBuilders.unshardedFinderBuilder()
                .withConnectionString("localhost:2181")
                .withNamespace("test")
                .withServiceName("test-service")
                .withDeserializer(new Deserializer<UnshardedClusterInfo>() {
                    @Override
                    public ServiceNode<UnshardedClusterInfo> deserialize(byte[] data) {
                        try {
                            return objectMapper.readValue(data,
                                    new TypeReference<ServiceNode<UnshardedClusterInfo>>() {
                                    });
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                        return null;
                    }
                })
                .build();
serviceFinder.start();
```

To find an instance:

```
ServiceNode node = serviceFinder.get(null); //null because you don't need to pass any shard info
//User node.hetHost() and node.getPort()
```

Stop the finder once you are done. (Generally this is when process ends)
```
serviceFinder.stop()
```

#### Finding an instance of a sharded Service Provider
This is similar to the above but for the type parameter you are using everywhere.

```
SimpleShardedServiceFinder<TestShardInfo> serviceFinder
    = ServiceFinderBuilders.<TestShardInfo>shardedFinderBuilder()
        .withConnectionString(testingCluster.getConnectString())
        .withNamespace("test")
        .withServiceName("test-service")
        .withDeserializer(new Deserializer<TestShardInfo>() {
            @Override
            public ServiceNode<TestShardInfo> deserialize(byte[] data) {
                try {
                    return objectMapper.readValue(data,
                            new TypeReference<ServiceNode<TestShardInfo>>() {
                            });
                } catch (IOException e) {
                    e.printStackTrace();
                }
                return null;
            }
        })
        .build();
serviceFinder.start();
```

Now you can find the service:

```
ServiceNode<TestShardInfo> node = serviceFinder.get(new TestShardInfo(1));
//Use host, port etc from the node
```

Stop the finder once you are done. (Generally this is when process ends)

```
serviceFinder.stop()
```

Version
----

0.3.0

Tech
-----------

Ranger uses Apache Curator:

* [Curator](http://curator.apache.org/) - Abstraction library for ZooKeeper operations


# [Dropwizard Service Discovery](https://github.com/santanusinha/dropwizard-service-discovery)

Provides service discovery to dropwizard services. It uses [Ranger](https://github.com/flipkart-incubator/ranger) for service discovery.

## Dependency for bundle
```
<dependency>
    <groupId>io.appform.dropwizard.discovery</groupId>
    <artifactId>dropwizard-service-discovery-bundle</artifactId>
    <version>1.3.13-2</version>
</dependency>
```

## Dependency for client
```
<dependency>
    <groupId>io.appform.dropwizard.discovery</groupId>
    <artifactId>dropwizard-service-discovery-client</artifactId>
    <version>1.3.13-1</version>
</dependency>
```

## How to use the bundle

You need to add an instance of type _ServiceDiscoveryConfiguration_ to your Dropwizard configuration file as follows:

```
public class AppConfiguration extends Configuration {
    //Your normal config
    @NotNull
    @Valid
    private ServiceDiscoveryConfiguration discovery = new ServiceDiscoveryConfiguration();
    
    //Whatever...
    
    public ServiceDiscoveryConfiguration getDiscovery() {
        return discovery;
    }
}
```

Next, you need to use this configuration in the Application while registering the bundle.

```
public class App extends Application<AppConfig> {
    private ServiceDiscoveryBundle<AppConfig> bundle;
    @Override
    public void initialize(Bootstrap<AppConfig> bootstrap) {
        bundle = new ServiceDiscoveryBundle<AppConfig>() {
            @Override
            protected ServiceDiscoveryConfiguration getRangerConfiguration(AppConfig appConfig) {
                return appConfig.getDiscovery();
            }

            @Override
            protected String getServiceName(AppConfig appConfig) {
                //Read from some config or hardcode your service name
                //This wi;l be used by clients to lookup instances for the service
                return "some-service";
            }

            @Override
            protected int getPort(AppConfig appConfig) {
                return 8080; //Parse config or hardcode
            }
        };
        
        bootstrap.addBundle(bundle);
    }

    @Override
    public void run(AppConfig configuration, Environment environment) throws Exception {
        ....
        //Register health checks
        bundle.registerHealthcheck(() -> {
                    //Check whatever
                    return HealthcheckStatus.healthy;
                });
        ...
    }
}
```
That's it .. your service will register to zookeeper when it starts up.

Sample config section might look like:
```
server:
  ...
  
discovery:
  namespace: mycompany
  environment: production
  zookeeper: "zk-server1.mycompany.net:2181,zk-server2.mycompany.net:2181"
  ...
  
...
```

The bundle also adds a jersey resource that lets you inspect the available instances.
Use GET /instances to see all instances that have been registered to your service.

## How to use the client
The client needs to be created and started. Once started it should never be stopped before the using service
itself dies or no queries will ever be made to ZK. Creation of the client is expensive.

```
ServiceDiscoveryClient client = ServiceDiscoveryClient.fromConnectionString()
                .connectionString("zk-server1.mycompany.net:2181, zk-server2.mycompany.net:2181")
                .namespace("mycompany")
                .serviceName("some-service")
                .environment("production")
                .objectMapper(new ObjectMapper())
                .build();
```

Start the client
```
client.start();
```

Get a valid node from the client:
```
Optional<ServiceNode<ShardInfo>> node = client.getNode();

//Always check if node is present for better stability
if(node.isPresent()) {
    //Use the endpoint details (host, port etc) 
    final String host = node.get().getHost();
    final int port = node.get().getPort();
    //Make service calls to the node etc etc
}
```

Close the client when not required:
```
client.stop();
```

*Note*
- Never save a node. The node query is extremely fast and does not make any remote calls.
- Repeat the above three times and follow it religiously.

## License
Apache 2

# NOTE
Package and group id has changed from `io.dropwizard.discovery` to `io.appfrom.dropwizard.discovery` from 1.3.12-2.

# [Dropwizard Riemann Bundle](https://github.com/phaneesh/riemann-bundle)

This bundle simplifies integrating dropwizard metrics with [Riemann](http://riemann.io/).
This bundle compiles only on Java 8.
 
## Dependencies
* [metrics3-riemann-reporter](https://github.com/riemann/riemann-java-client/tree/master/metrics3-riemann-reporter) 0.4.5 

## Usage
The bundle integrates dropwizard metrics with [Riemann](http://riemann.io/) with a simple configuration. 
 
### Build instructions
  - Clone the source:

        git clone github.com/phaneesh/riemann-bundle

  - Build

        mvn install

### Maven Dependency
Use the following repository:
```xml
<repository>
    <id>clojars</id>
    <name>Clojars repository</name>
    <url>https://clojars.org/repo</url>
</repository>
```
Use the following maven dependency:
```xml
<dependency>
    <groupId>io.raven.dropwizard</groupId>
    <artifactId>dropwizard-riemann</artifactId>
    <version>1.3.13-1</version>
</dependency>
```

### Using Riemann bundle

#### Bootstrap
```java
    @Override
    public void initialize(Bootstrap<?> bootstrap) {
        bootstrap.addBundle(new RiemannBundle() {
            
            @Override
            public RiemannConfig getRiemannConfiguration(Configuration configuration) {
                ...
            }
        });
    }
```

### Configuration
```
riemann:
  host: my.riemann.host
  port: 5556
  prefix: mycompany.myenvironment.myservice
  pollingInterval: 60 
  tags:
    - mytag1
    - mytag2
    - mytag3
```
# [Dropwizard DB Sharding Bundle](https://github.com/santanusinha/dropwizard-db-sharding-bundle)

Application level sharding for traditional relational databases.

See tests for details and code for documentation.

Apache Licensed

## Usage

The project dependencies are:
```
<dependency>
    <groupId>io.appform.dropwizard.sharding</groupId>
    <artifactId>db-sharding-bundle</artifactId>
    <version>1.3.13-4</version>
</dependency>
```
# NOTE
- Package and group id has changed from `io.dropwizard.sharding` to `io.appfrom.dropwizard.sharding` from 1.3.12-3.
- static create* methods have been replaced with instance methods from 1.3.13-4

# [FSM](https://github.com/koushikr/easy-fsm)

A small java state machine which would let you define events, states and their appropriate bindings.

> We are not saying that Evolution can't exist, only that it is guided by His Noodly Appendage.
>  - by Bobby Henderson, The Gospel of the Flying Spaghetti Monster

### Maven Dependency
Use the following repository:
```xml
<repository>
    <id>clojars</id>
    <name>Clojars repository</name>
    <url>https://clojars.org/repo</url>
</repository>
```
Use the following maven dependency:
```xml
<dependency>
    <groupId>io.github.fsm</groupId>
    <artifactId>fsm</artifactId>
    <version>0.0.1</version>
</dependency>
```

### Build instructions
  - Clone the source:

        git clone github.com/koushikr/easy-fsm

  - Build

        mvn install
        
# [Dropwizard RabbitMQ Actors Bundle](https://github.com/santanusinha/dropwizard-rabbitmq-actors)

Provides actor abstraction on RabbitMQ for dropwizard based projects.

## Dependency

```
<dependency>
    <groupId>io.appform.dropwizard.actors</groupId>
    <artifactId>dropwizard-rabbitmq-actors</artifactId>
    <version>1.3.13-1</version>
</dependency>
```

## License
Apache-2.0

# NOTE
Package and group id has changed from `io.dropwizard.actors` to `io.appfrom.dropwizard.actors` from 1.3.12-1.

# [QTrouper](https://github.com/koushikr/qtrouper)

> Do raindrops follow a certain hierarchy when they fall?
> - by Anthony T. Hincks


### Maven Dependency
Use the following repository:
```xml
<repository>
    <id>clojars</id>
    <name>Clojars repository</name>
    <url>https://clojars.org/repo</url>
</repository>
```
Use the following maven dependency:
```xml
<dependency>
    <groupId>io.github.qtrouper</groupId>
    <artifactId>qtrouper</artifactId>
    <version>0.0.1</version>
</dependency>
```

### Build instructions
  - Clone the source:

        git clone github.com/koushikr/qtrouper

  - Build

        mvn install

QTrouper provides the following capabilities
  - Create a hierarchy of retry and sideline for queue
  - Configure the number of times and the backoff on the retry queue
  - Enable consumption on the sideline queue or otherwise.
  - Queue the entire context with intermediate objects, so long as they are serializable.
  - An interface for consumption of main queue and sideline queue messages

# Key Concepts

  - Main Queue      : A queue on which the primary action is performed. Define a consumer on this queue by extending the Trouper class and implementing its process method.
  - Retry Queue     : A retry queue is not visible to the user. It is responsible for expiring a message and dead lettering into main queue till the maxRetryCount is not breached.
  - Sideline Queue  : A sideline queue on which the tertiary action is performed. Define a consumer on this queue by extending the Trouper class and implementing its processSideline method
  - QueueContex     : A Map to a key + a serializable object that gets enqueued in the queues
  - Trouper         : The Trouper is the queueing interface to provide main_queue, retry_queue and sideline_queue implementations for any call to action.


> The overriding design goal for qtrouper model
> is to make it as ready as possible to onboard a new message processor.
> The idea is that a formatted consumer definition is
> usable as-is, as a processing unit inside the app, without
> having to create any additiional boiler plate code
> to support the same. And a dropwizard bundle is made out of it
> for easy integration with dropwizard apps.

### Tech

QTrouper uses rabbitMQ as its backend interface and the

* [Dropwizard](https://github.com/dropwizard/dropwizard) - The bundle that got created
* [RabbitMQ](https://www.rabbitmq.com/) - Messaging that just works.

### Example

## Sample Configuration

```
static class SampleConfiguration extends Configuration{

        private RabbitConfiguration rabbitConfiguration;

    }
}

```

## Bundle Inclusion

```
      TrouperBundle<SampleConfiguration> trouperBundle = new TrouperBundle<SampleConfiguration>() {

                @Override
                public RabbitConfiguration getRabbitConfiguration(SampleConfiguration configuration) {
                    return null;
                }
      };

      bootstrap.addBundle(trouperBundle);

```

## Sample Actor

```

static class QueueingActor extends QTrouper<QueueContext> {

        @Inject
        public SymphonyActor(QueueConfiguration consumerConfiguration, RabbitConnection rabbitConnection) {
            super("sampleQueue",
                    consumerConfiguration,
                    rabbitConnection,
                    QueueContext.class,
                    Collections.emptySet());
        }

        @Override
        protected boolean process(QueueContext queueContext, QAccessInfo accessInfo){
            try{
                return processor.consume(queueContext, accessInfo);
            }catch (Exception e){
                log.error("Error processing a main queue message for reference Id {}", queueContext.getServiceReference());
                return false;
            }
        }

        @Override
        protected boolean processSideline(QueueContext queueContext, QAccessInfo accessInfo) {
            try{
                return processor.consumeSideline(queueContext, accessInfo);
            }catch (Exception e){
                log.error("Error processing a sideline queue message for reference Id {}", queueContext.getServiceReference());
                return false;
            }
        }
    }

```      
