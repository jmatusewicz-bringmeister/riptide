# Riptide: Failsafe

[![Valves](../docs/valves.jpg)](https://pixabay.com/en/wheel-valve-heating-line-turn-2137043/)

[![Build Status](https://img.shields.io/travis/zalando/riptide/master.svg)](https://travis-ci.org/zalando/riptide)
[![Coverage Status](https://img.shields.io/coveralls/zalando/riptide/master.svg)](https://coveralls.io/r/zalando/riptide)
[![Javadoc](https://www.javadoc.io/badge/org.zalando/riptide-failsafe.svg)](http://www.javadoc.io/doc/org.zalando/riptide-failsafe)
[![Release](https://img.shields.io/github/release/zalando/riptide.svg)](https://github.com/zalando/riptide/releases)
[![Maven Central](https://img.shields.io/maven-central/v/org.zalando/riptide-failsafe.svg)](https://maven-badges.herokuapp.com/maven-central/org.zalando/riptide-failsafe)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/zalando/riptide/master/LICENSE)

*Riptide: Failsafe* adds [Failsafe](https://github.com/jhalterman/failsafe) support to *Riptide*. It offers retries
and a circuit breaker to every remote call.

## Example

```java
Http.builder()
    .plugin(new FailsafePlugin(scheduler)
            .withRetryPolicy(retryPolicy)
            .withCircuitBreaker(circuitBreaker))
    .build();
```

## Features

- seamlessly integrates Riptide with Failsafe

## Dependencies

- Java 8
- Riptide Core
- Failsafe

## Installation

Add the following dependency to your project:

```xml
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>riptide-failsafe</artifactId>
    <version>${riptide.version}</version>
</dependency>
```

## Configuration

The failsafe plugin will not perform retries nor apply circuit breakers unless they were explicitly configured:

```java
Http.builder()
    .plugin(new FailsafePlugin(Executors.newScheduledThreadPool(20))
            .withRetryPolicy(new RetryPolicy()
                    .retryOn(SocketTimeoutException.class)
                    .withDelay(25, TimeUnit.MILLISECONDS)
                    .withMaxRetries(4))
            .withCircuitBreaker(new CircuitBreaker()
                    .withFailureThreshold(3, 10)
                    .withSuccessThreshold(5)
                    .withDelay(1, TimeUnit.MINUTES)))
    .build();
```

Please visit the [Failsafe readme](https://github.com/jhalterman/failsafe#readme) in order to see possible
configurations. Make sure you **check out 
[zalando/failsafe-actuator](https://github.com/zalando/failsafe-actuator)** for a seemless integration of
Failsafe and Spring Boot:

```java
@Autowired
private CircuitBreaker breaker;

Http.builder()
    .plugin(new FailsafePlugin(Executors.newScheduledThreadPool(20))
            .withRetryPolicy(new RetryPolicy()
                    .retryOn(SocketTimeoutException.class)
                    .withDelay(25, TimeUnit.MILLISECONDS)
                    .withMaxRetries(4))
            .withCircuitBreaker(breaker))
    .build();
```

## Usage

Given the failsafe plugin was configured as shown in the last section: A regular call like the following will now be
retried up to 4 times if the server did not respond within the socket timeout.

```java
http.get("/users/me")
    .dispatch(series(),
        on(SUCCESSFUL).call(User.class, this::greet),
        anySeries().call(problemHandling()))
```

Handling certain technical issues automatically, like socket timeouts, is quite useful.
But there might be cases where the server did respond, but the response indicates something that is worth
retrying, e.g. a `409 Conflict` or a `503 Service Unavailable`. Use the predefined `retry` route that comes with the
failsafe plugin:

```java
http.get("/users/me")
    .dispatch(series(),
        on(SUCCESSFUL).call(User.class, this::greet),
        on(CLIENT_ERROR).dispatch(status(),
            on(CONFLICT).call(retry())),
        on(SERVER_ERROR).dispatch(status(),
            on(SERVICE_UNAVAILABLE).call(retry())),
        anySeries().call(problemHandling()))
```

## Getting Help

If you have questions, concerns, bug reports, etc., please file an issue in this repository's [Issue Tracker](../../../../issues).

## Getting Involved/Contributing

To contribute, simply open a pull request and add a brief description (1-2 sentences) of your addition or change. For
more details, check the [contribution guidelines](../CONTRIBUTING.md).
