---
layout: post
title: "Eliminating 'if' Statements: Legacy Endpoint Primer"
description: "'if' statements tend to duplicate throughout the code base. This may lead to subtle mistakes and bugs. One way to avoid that problem is to eliminate 'if' statement completely. Today we are going to take a look at one example of such elimination."
date: 2016-08-20 11:59:37 +0200
comments: true
categories:
- kotlin
- code-smell
- clean-code
---

`if` statements tend to duplicate throughout the code base. This may lead to subtle mistakes and bugs. One way to avoid that problem is to eliminate `if` statement completely. Today we are going to take a look at one example of such elimination. Code examples today will be in Kotlin.

## Problem at hand

- Our API has an endpoint for issuing some sort of `verification token` given `device id` and `phone number` of the user's mobile device.
- We need to integrate these `verification tokens` with 3rd party API.
- The format of `verification token` is fairly standardized.
- After initial research, it turned out that `issuer` field of `verification token` has to be URL of the API that has issued that token and 3rd party API in question validates this fact.
- Currently, `issuer` field gets generated as `com.tddfellow`. According to this standard, it has to be `https://tddfellow.com`.
- Additionally, we have to support old versions of mobile clients for next 6 months, that are validating `issuer` to be `com.tddfellow`, we can not change them as they are already installed on users' mobile devices.

Solution: bump the version of our API from `v1` to `v2` and use `v1` for integration with old mobile clients and use `v2` for integration with 3rd party API and all new clients.

## Current Relevant Code

Main program, containing routing information:

```kotlin
// Main.kt

fun main(args: Array<String>) {
    val secureTokenSource = SimpleSecureTokenSource()
    val verificationTokenGateway = DummyVerificationTokenGateway()

    val issueVerificationTokenUseCase = UseCase(secureTokenSource, verificationTokenGateway)
    val issueVerificationTokenEndpoint = Endpoint(issueVerificationTokenUseCase)

    Spark.get("/api/v1/issueVerificationToken") { request, response ->
        issueVerificationTokenEndpoint.issueVerificationToken(
                deviceId = request.queryParams("deviceId"),
                phoneNumber = request.queryParams("phoneNumber")
        )
    }
}
```

Endpoint that issues verification token:

```kotlin
// ApiEndpoints/IssueVerificationToken/Endpoint.kt

class Endpoint(private val useCase: UseCase) {

    fun issueVerificationToken(deviceId: String, phoneNumber: String): IssueVerificationTokenEndpointResponse {
        val issueVerificationToken = useCase.issueVerificationToken(deviceId, phoneNumber)

        return IssueVerificationTokenEndpointResponse(
                issuer = issueVerificationToken.issuer,
                token = issueVerificationToken.secureToken
        )
    }

}
```

And the UseCase itself:

```kotlin
// IssueVerificationToken/UseCase.kt

open class UseCase(private val secureTokenSource: SecureTokenSource,
                   private val verificationTokenGateway: VerificationTokenGateway) {

    open fun issueVerificationToken(deviceId: String, phoneNumber: String): VerificationToken {
        val verificationToken = VerificationToken(
                issuer = "com.tddfellow",
                deviceId = deviceId,
                phoneNumber = phoneNumber,
                secureToken = secureTokenSource.generateToken()
        )

        verificationTokenGateway.persist(verificationToken)
        return verificationToken
    }

}
```

Code can be found [here](https://github.com/waterlink/LegacyEndpointPrimer/).

## Solution With Awkward `if` Statement

Easiest solution using passed in `apiVersion` from the `Main` program and switch on it being old or new in the use case to determine which issuer to generate:

```kotlin
// Main.kt

Spark.get("/api/v1/issueVerificationToken") { request, response ->
    issueVerificationTokenEndpoint.issueVerificationToken(
            deviceId = request.queryParams("deviceId"),
            phoneNumber = request.queryParams("phoneNumber"),

            // here we are passing "old" version to the endpoint
            apiVersion = "v1"
    )
}

Spark.get("/api/v2/issueVerificationToken") { request, response ->
    issueVerificationTokenEndpoint.issueVerificationToken(
            deviceId = request.queryParams("deviceId"),
            phoneNumber = request.queryParams("phoneNumber"),

            // here we are passing "new" version to the endpoint
            apiVersion = "v2"
    )
}
```

And the endpoint just passes this value through to the use case:

```kotlin
// ApiEndpoints/IssueVerificationToken/Endpoint.kt

fun issueVerificationToken(deviceId: String,
                           phoneNumber: String,
                           apiVersion: String): IssueVerificationTokenEndpointResponse {

    val issueVerificationToken = useCase.issueVerificationToken(
            deviceId = deviceId,
            phoneNumber = phoneNumber,

            // here we are passing API version through
            apiVersion = apiVersion
    )

    return IssueVerificationTokenEndpointResponse(
            issuer = issueVerificationToken.issuer,
            token = issueVerificationToken.secureToken
    )
}
```

And finally the `if` statement in the use case:

```kotlin
// IssueVerificationToken/UseCase.kt

private val OLD_ISSUER = "com.tddfellow"
private val NEW_ISSUER = "https://tddfellow.com"
private val OLD_API_VERSION = "v1"

fun issueVerificationToken(deviceId: String, phoneNumber: String, apiVersion: String): VerificationToken {
    val verificationToken = VerificationToken(
            issuer = getIssuerFor(apiVersion),
            deviceId = deviceId,
            phoneNumber = phoneNumber,
            secureToken = secureTokenSource.generateToken()
    )

    verificationTokenGateway.persist(verificationToken)
    return verificationToken
}

private fun getIssuerFor(apiVersion: String): String {
    if (apiVersion.equals(OLD_API_VERSION)) {
        return OLD_ISSUER
    }
    return NEW_ISSUER
}
```

The full code change available [here (via open Pull Request)](https://github.com/waterlink/LegacyEndpointPrimer/pull/1).

This solution has quite a few problems:

- `if` statement smells a bit
- use case probably should not have any knowledge of `apiVersion`, since APIs is not our domain, it is just a delivery mechanism

If we were to pass some object, like `TokenIssuer`, it would probably be more appropriate to have use case know of it. Let's try to refactor:

## Refactoring `if` Statement Using Polymorphism

First, let's start passing in the token issuer in the routing:

```kotlin
// Main.kt

Spark.get("/api/v1/issueVerificationToken") { request, response ->
    issueVerificationTokenEndpoint.issueVerificationToken(
            deviceId = request.queryParams("deviceId"),
            phoneNumber = request.queryParams("phoneNumber"),

            // here we pass a specific object now for old token issuer:
            tokenIssuer = OldTokenIssuer()
    )
}

Spark.get("/api/v2/issueVerificationToken") { request, response ->
    issueVerificationTokenEndpoint.issueVerificationToken(
            deviceId = request.queryParams("deviceId"),
            phoneNumber = request.queryParams("phoneNumber"),

            // here we pass an object that returns URL according to the standard
            tokenIssuer = UrlTokenIssuer()
    )
}
```

And this is how `TokenIssuer` and its derivatives are looking like:

```kotlin
// IssueVerificationToken/TokenIssuer.kt

interface TokenIssuer {

    fun getName(): String

}
```

```kotlin
// IssueVerificationToken/OldTokenIssuer.kt

class OldTokenIssuer : TokenIssuer {

    override fun getName() = "com.tddfellow"

}
```

```kotlin
// IssueVerificationToken/UrlTokenIssuer.kt

class UrlTokenIssuer : TokenIssuer {

    override fun getName() = "https://tddfellow.com"

}
```

As you might guess, endpoint just passes this object through to the use case. And the use case itself just calls `getName()` on it when generating issuer:

```kotlin
// IssueVerificationToken/UseCase.kt

fun issueVerificationToken(deviceId: String, phoneNumber: String, tokenIssuer: TokenIssuer): VerificationToken {
    val verificationToken = VerificationToken(

            // here is how much simpler it becomes
            issuer = tokenIssuer.getName(),

            deviceId = deviceId,
            phoneNumber = phoneNumber,
            secureToken = secureTokenSource.generateToken()
    )

    verificationTokenGateway.persist(verificationToken)
    return verificationToken
}
```

Full code change can be seen [here (in the Pull Request)](https://github.com/waterlink/LegacyEndpointPrimer/pull/2).

## Bottom Line

This code may be refactored further so that even `Endpoint` class will not have to know about `tokenIssuer` and pass it through. I will leave that as an exercise to you, my dear reader.

You would not want to miss next articles on this tech blog, we still have a lot to talk about:

- Continuous Integration and Continuous Delivery - importance of not impeding others,
- Open-Closed Principle - changing behavior by adding new code,
- Triangulation technique in Test-Driven Development - overlooking this technique might cause one fail at doing TDD,
- Mutational Testing, "Build Your Own Testing Framework" series, Test-Driven Development screencasts and so much more!

## Thanks!

Thank you for reading, my dear reader. If you liked it, please share this article on social networks and follow me on twitter: [@waterlink000](https://twitter.com/waterlink000).

If you have any questions or feedback for me, don't hesitate to reach me out on Twitter: [@waterlink000](https://twitter.com/waterlink000).
