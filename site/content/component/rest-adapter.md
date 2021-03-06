+++
title = "REST Adapter"
weight = 350
+++

The REST protocol adapter exposes a RESTful API for Eclipse Hono&trade;'s Telemetry and Event endpoints.
<!--more-->

The adapter is implemented as a Spring Boot application. It can be run either directly from the command line or by means of starting the corresponding Docker image created from it.

## Service Configuration

The following table provides an overview of the configuration variables and corresponding command line options for configuring the REST adapter.

| Environment Variable<br>Command Line Option | Mandatory | Default | Description |
| :------------------------------------------ | :-------: | :------ | :---------- |
| `HONO_APP_MAX_INSTANCES`<br>`--hono.app.maxInstances` | no | *#CPU cores* | The number of verticle instances to deploy. If not set, one verticle per processor core is deployed. |
| `HONO_APP_HEALTH_CHECK_PORT`<br>`--hono.app.healthCheckPort` | no | - | The port that the HTTP server, which exposes the service's health check resources, should bind to. If set, the adapter will expose a *readiness* probe at URI `/readiness` and a *liveness* probe at URI `/liveness`. |
| `HONO_APP_HEALTH_CHECK_BIND_ADDRESS`<br>`--hono.app.healthCheckBindAddress` | no | `127.0.0.1` | The IP address of the network interface that the HTTP server, which exposes the service's health check resources, should be bound to. The HTTP server will only be started if `HONO_APP_HEALTH_CHECK_BIND_ADDRESS` is set explicitly. |
| `HONO_HTTP_AUTHENTICATION_REQUIRED`<br>`--hono.http.authenticationRequired` | no | `true` | If set to `true` the protocol adapter requires devices to authenticate when connecting to the adapter. The credentials provided by the device are verified using the configured [Credentials Service]({{< relref "#credentials-service-connection-configuration" >}}). Devices that have failed to authenticate are not allowed to publish any data. |
| `HONO_HTTP_BIND_ADDRESS`<br>`--hono.http.bindAddress` | no | `127.0.0.1` | The IP address of the network interface that the secure port should be bound to.<br>See [Port Configuration]({{< relref "#port-configuration" >}}) below for details. |
| `HONO_HTTP_CERT_PATH`<br>`--hono.http.certPath` | no | - | The absolute path to the PEM file containing the certificate that the protocol adapter should use for authenticating to clients. This option must be used in conjunction with `HONO_HTTP_KEY_PATH`.<br>Alternatively, the `HONO_HTTP_KEY_STORE_PATH` option can be used to configure a key store containing both the key as well as the certificate. |
| `HONO_HTTP_INSECURE_PORT`<br>`--hono.http.insecurePort` | no | - | The insecure port the protocol adapter should listen on.<br>See [Port Configuration]({{< relref "#port-configuration" >}}) below for details. |
| `HONO_HTTP_INSECURE_PORT_BIND_ADDRESS`<br>`--hono.http.insecurePortBindAddress` | no | `127.0.0.1` | The IP address of the network interface that the insecure port should be bound to.<br>See [Port Configuration]({{< relref "#port-configuration" >}}) below for details. |
| `HONO_HTTP_INSECURE_PORT_ENABLED`<br>`--hono.http.insecurePortEnabled` | no | `false` | If set to `true` the protocol adapter will open an insecure port (not secured by TLS) using either the port number set via `HONO_HTTP_INSECURE_PORT` or the default port number (`8080`) if not set explicitly.<br>See [Port Configuration]({{< relref "#port-configuration" >}}) below for details. |
| `HONO_HTTP_KEY_PATH`<br>`--hono.http.keyPath` | no | - | The absolute path to the (PKCS8) PEM file containing the private key that the protocol adapter should use for authenticating to clients. This option must be used in conjunction with `HONO_HTTP_CERT_PATH`. Alternatively, the `HONO_HTTP_KEY_STORE_PATH` option can be used to configure a key store containing both the key as well as the certificate. |
| `HONO_HTTP_KEY_STORE_PASSWORD`<br>`--hono.http.keyStorePassword` | no | - | The password required to read the contents of the key store. |
| `HONO_HTTP_KEY_STORE_PATH`<br>`--hono.http.keyStorePath` | no | - | The absolute path to the Java key store containing the private key and certificate that the protocol adapter should use for authenticating to clients. Either this option or the `HONO_HTTP_KEY_PATH` and `HONO_HTTP_CERT_PATH` options need to be set in order to enable TLS secured connections with clients. The key store format can be either `JKS` or `PKCS12` indicated by a `.jks` or `.p12` file suffix respectively. |
| `HONO_HTTP_MAX_PAYLOAD_SIZE`<br>`--hono.http.maxPayloadSize` | no | `2048` | The maximum allowed size of an incoming HTTP request's body in bytes. Requests with a larger body size are rejected with a 413 `Request entity too large` response. |
| `HONO_HTTP_PORT`<br>`--hono.http.port` | no | `8443` | The secure port that the protocol adapter should listen on.<br>See [Port Configuration]({{< relref "#port-configuration" >}}) below for details. |
| `HONO_HTTP_REALM`<br>`--hono.http.realm` | no | `Hono` | The name of the *realm* that unauthenticated devices are prompted to provide credentials for. The realm is used in the *WWW-Authenticate* header returned to devices in response to unauthenticated requests. |
| `HONO_HTTP_REG_ASSERTION_ENABLED`<br>`--hono.http.regAssertionEnabled` | no | `false` | If set to `true` the protocol adapter includes a Java Web Token in the *Hono-Reg-Assertion* header of the HTTP response to *publish* requests which asserts the device's registration status. The device MAY include this token in subsequent requests to prevent the adapter from requesting a fresh assertion from the *Device Registration* service on each invocation. This will reduce the overall latency of the request. However, results may vary depending on the deployment scenario. |

The variables only need to be set if the default value does not match your environment.


## Port Configuration

The REST protocol adapter can be configured to listen for connections on

* a secure port only (default) or
* an insecure port only or
* both a secure and an insecure port (dual port configuration)

The REST protocol adapter will fail to start if none of the ports is configured properly.

### Secure Port Only

The protocol adapter needs to be configured with a private key and certificate in order to open a TLS secured port.

There are two alternative ways for doing so:

1. Setting the `HONO_HTTP_KEY_STORE_PATH` and the `HONO_HTTP_KEY_STORE_PASSWORD` variables in order to load the key & certificate from a password protected key store, or
1. setting the `HONO_HTTP_KEY_PATH` and `HONO_HTTP_CERT_PATH` variables in order to load the key and certificate from two separate PEM files in PKCS8 format.

When starting up, the protocol adapter will bind a TLS secured socket to the default secure port 8443. The port number can also be set explicitly using the `HONO_HTTP_PORT` variable.

The `HONO_HTTP_BIND_ADDRESS` variable can be used to specify the network interface that the port should be exposed on. By default the port is bound to the *loopback device* only, i.e. the port will only be accessible from the local host. Setting this variable to `0.0.0.0` will let the port being bound to **all** network interfaces (be careful not to expose the port unintentionally to the outside world).

### Insecure Port Only

The secure port will mostly be required for production scenarios. However, it might be desirable to expose a non-TLS secured port instead, e.g. for testing purposes. In any case, the non-secure port needs to be explicitly enabled either by

- explicitly setting `HONO_HTTP_INSECURE_PORT` to a valid port number, or by
- implicitly configuring the default port (8080) by simply setting `HONO_HTTP_INSECURE_PORT_ENABLED` to `true`.

The protocol adapter issues a warning on the console if `HONO_HTTP_INSECURE_PORT` is set to the default secure HTTP port (8443).

The `HONO_HTTP_INSECURE_PORT_BIND_ADDRESS` variable can be used to specify the network interface that the port should be exposed on. By default the port is bound to the *loopback device* only, i.e. the port will only be accessible from the local host. This variable might be used to e.g. expose the non-TLS secured port on a local interface only, thus providing easy access from within the local network, while still requiring encrypted communication when accessed from the outside over public network infrastructure.

Setting this variable to `0.0.0.0` will let the port being bound to **all** network interfaces (be careful not to expose the port unintentionally to the outside world).

### Dual Port
 
The protocol adapter may be configured to open both a secure and a non-secure port at the same time simply by configuring both ports as described above. For this to work, both ports must be configured to use different port numbers, otherwise startup will fail.

### Ephemeral Ports

Both the secure as well as the insecure port numbers may be explicitly set to `0`. The protocol adapter will then use arbitrary (unused) port numbers determined by the operating system during startup.


## Hono Messaging Connection Configuration

The adapter requires a connection to the Hono Messaging component in order to forward telemetry data and events received from devices to downstream consumers.

The following table provides an overview of the configuration variables and corresponding command line options for configuring the connection to the Hono Messaging component.

| Environment Variable<br>Command Line Option | Mandatory | Default Value | Description  |
| :------------------------------------------ | :-------: | :------------ | :------------|
| `HONO_MESSAGING_HOST`<br>`--hono.messaging.host` | yes | `localhost` | The IP address or name of the Hono Messaging host to connect to. NB: This needs to be set to an address that can be resolved within the network the adapter runs on. When running as a Docker container, use Docker's `--network` command line option to attach the adapter container to the Docker network that the *Hono Server* container is running on. |
| `HONO_MESSAGING_PORT`<br>`--hono.messaging.port` | yes | `5671` | The port that the Hono Messaging component is listening on. |
| `HONO_MESSAGING_USERNAME`<br>`--hono.messaging.username` | yes | - | The username to use for authenticating to the Hono Messaging component. |
| `HONO_MESSAGING_PASSWORD`<br>`--hono.messaging.password` | yes | - | The password to use for authenticating to the Hono Messaging component. |
| `HONO_MESSAGING_TRUST_STORE_PATH`<br>`--hono.messaging.trustStorePath` | no  | - | The absolute path to the Java key store containing the CA certificates the adapter uses for authenticating the Hono Messaging component. This property **must** be set if the Hono Messaging component has been configured to support TLS. The key store format can be either `JKS`, `PKCS12` or `PEM` indicated by a `.jks`, `.p12` or `.pem` file suffix respectively. |
| `HONO_MESSAGING_TRUST_STORE_PASSWORD`<br>`--hono.messaging.trustStorePassword` | no | - | The password required to read the contents of the trust store. |

## Device Registration Service Connection Configuration

The adapter requires a connection to an implementation of Hono's Device Registration API in order to retrieve registration status assertions for connected devices.

The following table provides an overview of the configuration variables and corresponding command line options for configuring the connection to the Device Registration service.

| Environment Variable<br>Command Line Option | Mandatory | Default Value | Description  |
| :------------------------------------------ | :-------: | :------------ | :------------|
| `HONO_REGISTRATION_HOST`<br>`--hono.registration.host` | yes | `localhost` | The IP address or name of the Device Registration service. The adapter uses this service to get an assertion regarding a device's registration status, i.e. whether it is enabled and if it is registered with a particular tenant. NB: This variable needs to be set to an address that can be resolved within the network the adapter runs on. When running as a Docker container, use Docker's `--network` command line option to attach the adapter container to the same network the Device Registration service container is running on. |
| `HONO_REGISTRATION_PORT`<br>`--hono.registration.port` | yes | `5671` | The port that the Device Registration service is listening on. |
| `HONO_REGISTRATION_USERNAME`<br>`--hono.registration.username` | yes | - | The username to use for authenticating to the Device Registration service. |
| `HONO_REGISTRATION_PASSWORD`<br>`--hono.registration.password` | yes | - | The password to use for authenticating to the Device Registration service. |
| `HONO_REGISTRATION_TRUST_STORE_PATH`<br>`--hono.registration.trustStorePath` | no  | - | The absolute path to the Java key store containing the CA certificates the adapter uses for authenticating the Device Registration service. This property **must** be set if the Device Registration service has been configured to use TLS. The key store format can be either `JKS`, `PKCS12` or `PEM` indicated by a `.jks`, `.p12` or `.pem` file suffix. |
| `HONO_REGISTRATION_TRUST_STORE_PASSWORD`<br>`--hono.registration.trustStorePassword` | no | - | The password required to read the contents of the trust store. |

## Credentials Service Connection Configuration

The adapter requires a connection to an implementation of Hono's Credentials API in order to retrieve credentials stored for devices that need to be authenticated.

The following table provides an overview of the configuration variables and corresponding command line options for configuring the connection to the Credentials service.

| Environment Variable<br>Command Line Option | Mandatory | Default Value | Description  |
| :------------------------------------------ | :-------: | :------------ | :------------|
| `HONO_CREDENTIALS_HOST`<br>`--hono.credentials.host` | yes | `localhost` | The IP address or name of the Credentials service. The adapter uses this service to get credentials stored for a device to authenticate it. NB: This variable needs to be set to an address that can be resolved within the network the adapter runs on. When running as a Docker container, use Docker's `--network` command line option to attach the adapter container to the same network the Credentials service container is running on. |
| `HONO_CREDENTIALS_PORT`<br>`--hono.credentials.port` | yes | `5671` | The port that the Credentials service is listening on. |
| `HONO_CREDENTIALS_USERNAME`<br>`--hono.credentials.username` | yes | - | The username to use for authenticating to the Credentials service. |
| `HONO_CREDENTIALS_PASSWORD`<br>`--hono.credentials.password` | yes | - | The password to use for authenticating to the Credentials service. |
| `HONO_CREDENTIALS_TRUST_STORE_PATH`<br>`--hono.credentials.trustStorePath` | no  | - | The absolute path to the Java key store containing the CA certificates the adapter uses for authenticating the Credentials service. This property **must** be set if the Credentials service has been configured to use TLS. The key store format can be either `JKS`, `PKCS12` or `PEM` indicated by a `.jks`, `.p12` or `.pem` file suffix. |
| `HONO_CREDENTIALS_TRUST_STORE_PASSWORD`<br>`--hono.credentials.trustStorePassword` | no | - | The password required to read the contents of the trust store. |


## Run as a Docker Swarm Service

The REST adapter can be run as a Docker container from the command line. The following commands create and start the REST adapter as a Docker Swarm service using the default keys  contained in the `demo-certs` module:

~~~sh
~/hono$ docker secret create trusted-certs.pem demo-certs/certs/trusted-certs.pem
~/hono$ docker secret create rest-adapter-key.pem demo-certs/certs/rest-adapter-key.pem
~/hono$ docker secret create rest-adapter-cert.pem demo-certs/certs/rest-adapter-cert.pem
~/hono$ docker service create --detach --name hono-adapter-rest-vertx --network hono-net -p 8080:8080 -p 8443:8443  \
> --secret trusted-certs.pem \
> --secret rest-adapter-key.pem \
> --secret rest-adapter-cert.pem \
> -e 'HONO_MESSAGING_HOST=hono-service-messaging.hono' \
> -e 'HONO_MESSAGING_USERNAME=rest-adapter@HONO' \
> -e 'HONO_MESSAGING_PASSWORD=rest-secret' \
> -e 'HONO_MESSAGING_TRUST_STORE_PATH=/run/secrets/trusted-certs.pem' \
> -e 'HONO_REGISTRATION_HOST=hono-service-device-registry.hono' \
> -e 'HONO_REGISTRATION_USERNAME=rest-adapter@HONO' \
> -e 'HONO_REGISTRATION_PASSWORD=rest-secret' \
> -e 'HONO_REGISTRATION_TRUST_STORE_PATH=/run/secrets/trusted-certs.pem' \
> -e 'HONO_HTTP_BIND_ADDRESS=0.0.0.0' \
> -e 'HONO_HTTP_KEY_PATH=/run/secrets/rest-adapter-key.pem' \
> -e 'HONO_HTTP_CERT_PATH=/run/secrets/rest-adapter-cert.pem' \
> -e 'HONO_HTTP_INSECURE_PORT_ENABLED=true' \
> -e 'HONO_HTTP_INSECURE_PORT_BIND_ADDRESS=0.0.0.0'
> eclipsehono/hono-adapter-rest-vertx:latest
~~~

{{% note %}}
There are several things noteworthy about the above command to start the service:

1. The *secrets* need to be created once only, i.e. they only need to be removed and re-created if they are changed.
1. The *--network* command line switch is used to specify the *user defined* Docker network that the REST adapter container should attach to. It is important that the REST adapter container is attached to the same network that the Hono Messaging component is attached to so that the REST adapter can use the Hono Messaging component's host name to connect to it via the Docker network. Please refer to the [Docker Networking Guide](https://docs.docker.com/engine/userguide/networking/#/user-defined-networks) for details regarding how to create a *user defined* network in Docker.
1. In cases where the REST adapter container requires a lot of configuration via environment variables (provided by means of *-e* switches), it is more convenient to add all environment variable definitions to a separate *env file* and refer to it using Docker's *--env-file* command line switch when starting the container. This way the command line to start the container is much shorter and can be copied and edited more easily.
{{% /note %}}

## Run using the Docker Swarm Deployment Script

In most cases it is much easier to start all of Hono's components in one shot using the Docker Swarm deployment script provided in the `example/target/deploy/docker` folder.

## Run the Spring Boot Application

Sometimes it is helpful to run the adapter from its jar file, e.g. in order to attach a debugger more easily or to take advantage of code replacement.
In order to do so, the adapter can be started using the `spring-boot:run` maven goal from the `adapters/rest-vertx` folder.
The corresponding command to start up the adapter with the configuration used in the Docker example above looks like this:

~~~sh
~/hono/adapters/rest-vertx$ mvn spring-boot:run -Drun.arguments=\
> --hono.messaging.host=hono-service-messaging.hono,\
> --hono.messaging.username=rest-adapter@HONO,\
> --hono.messaging.password=rest-secret,\
> --hono.messaging.trustStorePath=target/certs/trusted-certs.pem \
> --hono.registration.host=hono-service-device-registry.hono,\
> --hono.registration.username=rest-adapter@HONO,\
> --hono.registration.password=rest-secret,\
> --hono.registration.trustStorePath=target/certs/trusted-certs.pem \
> --hono.http.bindAddress=0.0.0.0 \
> --hono.http.insecurePortEnabled=true,\
> --hono.http.insecurePortBindAddress=0.0.0.0
~~~

{{% note %}}
In the example above the *--hono.messaging.host=hono-service-messaging.hono* command line option indicates that the Hono Messaging component is running on a host
with name *hono-service-messaging.hono*. However, if the Hono Messaging component has been started as a Docker container then the *hono-service-messaging.hono* host name will most likely only be resolvable on the network that Docker has created for running the container on, i.e. when you run the REST adapter
from the Spring Boot application and want it to connect to a Hono Messaging instance run as a Docker container then you need to set the
value of the *--hono.messaging.host* option to the IP address (or name) of the Docker host running the Hono Messaging container.
The same holds true analogously for the *hono-service-device-registry.hono* address.
{{% /note %}}

## Using the API

The REST adapter by default requires devices to authenticate during connection establishment. The adapter supports the [Basic HTTP authentication scheme](https://tools.ietf.org/html/rfc7617). The *username* provided in the header must have the form *auth-id@tenant*, e.g. `sensor1@DEFAULT_TENANT`. The adapter verifies the credentials provided by the device against the credentials that the [configured Credentials service]({{< relref "#credentials-service-configuration" >}}) has on record for the device. The adapter uses the Credentials API's *get* operation to retrieve the credentials-on-record with the *tenant* and *auth-id* provided by the device in the *username* and `hashed-password` as the *type* of secret as query parameters.

When running the Hono example installation as described in the [Getting Started guide]({{< relref "getting-started.md" >}}), the demo Credentials service comes pre-configured with a `hashed-password` secret for device `4711` of tenant `DEFAULT_TENANT` having an *auth-id* of `sensor1` and (hashed) *password* `hono-secret`. These credentials are used in the following examples illustrating the usage of the adapter. Please refer to the [Credentials API]({{< relref "api/Credentials-API.md#standard-credential-types" >}}) for details regarding the different types of secrets.

{{% note %}}
There is a subtle difference between the *device identifier* (*device-id*) and the *auth-id* a device uses for authentication. See [Device Identity]({{< relref "concepts/device-identity.md" >}}) for a discussion of the concepts.
{{% /note %}}

### Publish Telemetry Data

* URI: `/telemetry`
* Method: `POST`
* Request Headers:
  * (required) `Authorization`: The device's *auth-id* and plain text password encoded according to the [Basic HTTP authentication scheme](https://tools.ietf.org/html/rfc7617).
  * (required) `Content-Type`: The type of payload contained in the body.
  * (optional) `Hono-Reg-Assertion`: A JSON Web Token asserting the device's registration status (see [assert Device Registration]({{< relref "api/Device-Registration-API.md#assert-device-registration" >}})).
* Request Body:
  * (required) Arbitrary payload encoded according to the given content type.
* Status Codes:
  * 202 (Accepted): The telemetry data has been accepted for processing. Note that this does not *guarantee* successful delivery to potential consumers.
  * 400 (Bad Request): The request cannot be processed because the content type header is missing or the request body is empty.
  * 401 (Unauthorized): The request cannot be processed because the request does not contain valid credentials.
  * 403 (Forbidden): The request cannot be processed because the device's registration status cannot be asserted, i.e. the given device either does not belong to the given tenant or is disabled.
  * 503 (Service Unavailable): The request cannot be processed because there is no consumer of telemetry data for the given tenant connected to Hono.
* Response Headers:
  * (optional) `Hono-Reg-Assertion`: A JSON Web Token asserting the device's registration status (see [assert Device Registration]({{< relref "api/Device-Registration-API.md#assert-device-registration" >}})). A client SHOULD include this token on subsequent telemetry or event requests for the same device in order to prevent the REST adapter from requesting a fresh assertion from the *Device Registration* service on each invocation. This header will be included in a response every time a new token has been issued. Note that this header will only be generated if the `HONO_HTTP_REG_ASSERTION_ENABLED` property is explicitly set to `true`.

This is the preferred way for devices to publish telemetry data. It is available only if the protocol adapter is configured to require devices to authenticate (which is the default).

**Examples**

Publish some JSON data for device `4711`:

    $ curl -i -X POST -u sensor1@DEFAULT_TENANT:hono-secret -H 'Content-Type: application/json' \
    $ --data-binary '{"temp": 5}' http://127.0.0.1:8080/telemetry

Response:

    HTTP/1.1 202 Accepted
    Content-Length: 0

### Publish Telemetry Data (unauthenticated devices)

* URI: `/telemetry/${tenantId}/${deviceId}`
* Method: `PUT`
* Request Headers:
  * (optional) `Authorization`: The device's *auth-id* and plain text password encoded according to the [Basic HTTP authentication scheme](https://tools.ietf.org/html/rfc7617).
  * (required) `Content-Type`: The type of payload contained in the body.
  * (optional) `Hono-Reg-Assertion`: A JSON Web Token asserting the device's registration status (see [assert Device Registration]({{< relref "api/Device-Registration-API.md#assert-device-registration" >}})) and its identity.
* Request Body:
  * (required) Arbitrary payload encoded according to the given content type.
* Status Codes:
  * 202 (Accepted): The telemetry data has been accepted for processing. Note that this does not *guarantee* successful delivery to potential consumers.
  * 400 (Bad Request): The request cannot be processed because the content type header is missing or the request body is empty.
  * 403 (Forbidden): The request cannot be processed because the device's registration status cannot be asserted, i.e. the given device either does not belong to the given tenant or is disabled.
  * 503 (Service Unavailable): The request cannot be processed because there is no consumer of telemetry data for the given tenant connected to Hono.
* Response Headers:
  * (optional) `Hono-Reg-Assertion`: A JSON Web Token asserting the device's registration status (see [assert Device Registration]({{< relref "api/Device-Registration-API.md#assert-device-registration" >}})). A client SHOULD include this token on subsequent telemetry or event requests for the same device in order to prevent the REST adapter from requesting a fresh assertion from the *Device Registration* service on each invocation. This header will be included in a response every time a new token has been issued. Note that this header will only be generated if the `HONO_HTTP_REG_ASSERTION_ENABLED` property is explicitly set to `true`.

This URI MUST be used by devices that have not authenticated to the protocol adapter. Note that this requires the `HONO_HTTP_AUTHENTICATION_REQUIRED` configuration property to be explicitly set to `false`.

For reasons of completeness, this URI MAY also be used by devices that do have authenticated to the protocol adapter. In this case the protocol adapter verifies that the values provided in the path parameters match the credentials that the device has provided during authentication. If they do not match, the adapter responds with a 403 status code.

**Examples**

Publish some JSON data for device `4711`:

    $ curl -i -X PUT -H 'Content-Type: application/json' \
    $ --data-binary '{"temp": 5}' http://127.0.0.1:8080/telemetry/DEFAULT_TENANT/4711

Response:

    HTTP/1.1 202 Accepted
    Content-Length: 0

An authenticated device may also use this URI. In this case the device needs to know both the *auth-id* as well as the corresponding *device identifier*:

    $ curl -i -X PUT -u sensor1@DEFAULT_TENANT:hono-secret -H 'Content-Type: application/json' \
    $ --data-binary '{"temp": 5}' http://127.0.0.1:8080/telemetry/DEFAULT_TENANT/4711

In real world scenarios this would usually only make sense if the *device identifier* is also used as the *auth-id*, i.e.

    $ curl -i -X PUT -u 4711@DEFAULT_TENANT:hono-secret -H 'Content-Type: application/json' \
    $ --data-binary '{"temp": 5}' http://127.0.0.1:8080/telemetry/DEFAULT_TENANT/4711

### Publish an Event

* URI: `/event`
* Method: `POST`
* Request Headers:
  * (required) `Authorization`: The device's *auth-id* and plain text password encoded according to the [Basic HTTP authentication scheme](https://tools.ietf.org/html/rfc7617).
  * (required) `Content-Type`: The type of payload contained in the body.
  * (optional) `Hono-Reg-Assertion`: A JSON Web Token asserting the device's registration status (see [assert Device Registration]({{< relref "api/Device-Registration-API.md#assert-device-registration" >}})) and its identity. Including this header only makes sense if the adapter's `HONO_HTTP_REG_ASSERTION_ENABLED` property has been set to `true`.
* Request Body:
  * (required) Arbitrary payload encoded according to the given content type.
* Status Codes:
  * 202 (Accepted): The telemetry data has been accepted for processing. Note that this does not *guarantee* successful delivery to potential consumers.
  * 400 (Bad Request): The request cannot be processed because the content type header is missing or the request body is empty.
  * 401 (Unauthorized): The request cannot be processed because the request does not contain valid credentials.
  * 403 (Forbidden): The request cannot be processed because the device's registration status cannot be asserted, i.e. the given device either does not belong to the given tenant or is disabled.
  * 503 (Service Unavailable): The request cannot be processed because there is no consumer of events for the given tenant connected to Hono.
* Response Headers:
  * (optional) `Hono-Reg-Assertion`: A JSON Web Token asserting the device's registration status (see [assert Device Registration]({{< relref "api/Device-Registration-API.md#assert-device-registration" >}})). A client MAY include this token on subsequent telemetry or event requests for the same device in order to prevent the adapter from requesting a fresh assertion from the *Device Registration* service on each invocation. This header will be included in a response every time a new token has been issued. Note that this header will only be generated if the `HONO_HTTP_REG_ASSERTION_ENABLED` property is explicitly set to `true`.

This is the preferred way for devices to publish events. It is available only if the protocol adapter is configured to require devices to authenticate (which is the default).

**Example**

Publish some JSON data for device `4711`:

    $ curl -i -X POST -u sensor1@DEFAULT_TENANT:hono-secret -H 'Content-Type: application/json' \
    $ --data-binary '{"alarm": true}' http://127.0.0.1:8080/event

Response:

    HTTP/1.1 202 Accepted
    Content-Length: 0

### Publish an Event (unauthenticated devices)

* URI: `/event/${tenantId}/${deviceId}`
* Method: `PUT`
* Request Headers:
  * (optional) `Authorization`: The device's *auth-id* and plain text password encoded according to the [Basic HTTP authentication scheme](https://tools.ietf.org/html/rfc7617).
  * (required) `Content-Type`: The type of payload contained in the body.
  * (optional) `Hono-Reg-Assertion`: A JSON Web Token asserting the device's registration status (see [assert Device Registration]({{< relref "api/Device-Registration-API.md#assert-device-registration" >}}))
* Request Body:
  * (required) Arbitrary payload encoded according to the given content type.
* Status Codes:
  * 202 (Accepted): The telemetry data has been accepted for processing. Note that this does not *guarantee* successful delivery to potential consumers.
  * 400 (Bad Request): The request cannot be processed because the content type header is missing or the request body is empty.
  * 403 (Forbidden): The request cannot be processed because the device's registration status cannot be asserted, i.e. the given device either does not belong to the given tenant or is disabled.
  * 503 (Service Unavailable): The request cannot be processed because there is no consumer of events for the given tenant connected to Hono.
* Response Headers:
  * (optional) `Hono-Reg-Assertion`: A JSON Web Token asserting the device's registration status (see [assert Device Registration]({{< relref "api/Device-Registration-API.md#assert-device-registration" >}})). A client SHOULD include this token on subsequent telemetry or event requests for the same device in order to prevent the REST adapter from requesting a fresh assertion from the *Device Registration* service on each invocation. This header will be included in a response every time a new token has been issued. Note that this header will only be generated if the `HONO_HTTP_REG_ASSERTION_ENABLED` property is explicitly set to `true`.

This URI MUST be used by devices that have not authenticated to the protocol adapter. Note that this requires the `HONO_HTTP_AUTHENTICATION_REQUIRED` configuration property to be explicitly set to `false`.

For reasons of completeness, this URI MAY also be used by devices that do have authenticated to the protocol adapter. In this case the protocol adapter verifies that the values provided in the path parameters match the credentials that the device has provided during authentication. If they do not match, the adapter responds with a 403 status code.

**Examples**

Publish some JSON data for device `4711`:

    $ curl -i -X PUT -H 'Content-Type: application/json' \
    $ --data-binary '{"alarm": true}' http://127.0.0.1:8080/event/DEFAULT_TENANT/4711

Response:

    HTTP/1.1 202 Accepted
    Content-Length: 0

An authenticated device may also use this URI. In this case the device needs to know both the *auth-id* as well as the corresponding *device identifier*:

    $ curl -i -X PUT -u sensor1@DEFAULT_TENANT:hono-secret -H 'Content-Type: application/json' \
    $ --data-binary '{"alarm": true}' http://127.0.0.1:8080/event/DEFAULT_TENANT/4711

In real world scenarios this would usually only make sense if the *device identifier* is also used as the *auth-id*, i.e.

    $ curl -i -X PUT -u 4711@DEFAULT_TENANT:hono-secret -H 'Content-Type: application/json' \
    $ --data-binary '{"alarm": true}' http://127.0.0.1:8080/event/DEFAULT_TENANT/4711
