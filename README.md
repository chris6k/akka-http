# Akka Http   [![Build Status](https://secure.travis-ci.org/thenewmotion/akka-http.png)](http://travis-ci.org/thenewmotion/akka-http)

<table border="0">
  <tr>
    <td>Scala version: </td>
    <td>2.10.1</td>
  </tr>
  <tr>
    <td>Akka version: </td>
    <td>2.1.2</td>
  </tr>
</table>

Akka Http is not a webframework. It's an Akka based implementation of asynchronous requests available since servlet 3.0
Each request will have own actor. Actor will live until async request is processed and completed or time is out.
Processing of async request is divided by different unit of work. All these work units will be processed one by one, however
Actor may receive one of the javax.servlet.AsyncEvent between. In case of Error, Timeout or Complete events actor will stop processing and shutdown.
So all work units left will be dropped.

# Examples

[HelloWorldServlet example](https://github.com/thenewmotion/akka-http-helloworld)

# What's new

## Scopes

Async context has been divided on different scopes. Each scope is a synchronous unit of work

* Request scope. Expected to be the heaviest scope. All request related logic should go here

* Response scope. Light scope for putting request result in response

We need scope to receive async event messages in between. Thus will allow us to avoid various exceptions
For example you may get `response time out exception` or `response already completed`, etc.
Also you can avoid odd operations when requrest has expired.

# Migration Guide


## RequestMethod

RequestMethod was removed. However you are able to use HttpServletRequest/Response directly.

* Using old akka 1.3 akka-mist

```scala
  def receive = {
    case get: Get =>
      get.response.setContentType("text/html")

      val html =
        <html>
          <body>
            <h1>Hello World</h1>
            <h3>endpoint actor</h3>
          </body>
        </html>

      get.OK(html)
  }
```

* Using new akka 2.0 akka-http

```scala
  def receive = {
    case req: HttpServletRequest =>

      // doing some heavy work here

      //will be called for completing request
      val future = FutureResponse {
        res =>
          res.getWriter.write(
            <html>
              <body>
                <h1>Hello World</h1>
                <h3>endpoint actor</h3>
              </body>
            </html>.toString())
          res.getWriter.close()
      }

      //passing `future` to AsyncActor, created for this AsyncContext
      sender ! Complete(future)
  }
```


## Endpoints

We are not using Endpoint hierarchy from akka-mist. We have single `EndpointsAgent(akka.agent)` instead where you can find/attach/detach endpoint in the runtime.

## MistSettings

MistSettings were moved to HttpExtension with other common stuff. Http properties will be defined per ActorSystem.

## AkkaMistServlet/AkkaMistFilter

AkkaMistServlet was renamed to AkkaHttpServlet, AkkaMistFilter to AkkaHttpFilter. Both of them extend trait AkkaHttp

## Configuration

From [Akka Migration Guide](http://doc.akka.io/docs/akka/2.0/project/migration-guide-1.3.x-2.0.x.html#Configuration)

`In v1.3 the default name of the configuration file was akka.conf. In v2.0 the default name is application.conf.`

Akka Http properties has been removed from `akka-reference.conf (akka.conf)`.
So Akka Mist has own `reference.conf`.
However some of the properties were removed from mist at all, same happened with related functionality.
Most of the properties you can retrieve from `HttpExtension.scala`.
[Extension](http://doc.akka.io/docs/akka/2.0/scala/extending-akka.html) mechanism is new, added in Akka 2.0

### Added properties: ###

* `system-name = http` - name of the system used in servlet
* `log-http-config-on-init = off` - Log akka.http configuration at INFO level on Servlet/Filter init.

### Removed properties: ###

* `root-actor-id = "_httproot"`
* `hostname = "localhost"`
* `port = 9998`
* `filters = ["akka.security.AkkaSecurityFilterFactory"]`
* `resource-packages = []`
* `authenticator = "N/A"`
* `connection-close = true`
* `kerberos.servicePrincipal = "N/A"`
* `kerberos.keyTabLocation = "N/A"`
* `kerberos.kerberosDebug = "N/A"`
* `kerberos.realm = ""`

## Setup

1. Add this repository to your pom.xml:
```xml
    <repository>
        <id>thenewmotion</id>
        <name>The New Motion Repository</name>
        <url>http://nexus.thenewmotion.com/content/repositories/releases-public</url>
    </repository>
```

2. Add dependency to your pom.xml:
```xml
    <dependency>
        <groupId>com.thenewmotion.akka</groupId>
        <artifactId>akka-http_2.10</artifactId>
        <version>1.0.3</version>
    </dependency>
```
