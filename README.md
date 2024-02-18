# NetArmor
##### Where Security Meets Innovation.

## Introduction
**NetArmor** is a <b>reactive</b> enterprise security solution designed for today’s dynamic digital environments. Blending the best of ESAPI and OWASP Top 10, NetArmor is tailored for Netty-based applications, providing a robust defense layer for high-risk sectors, including finance.

Built for scalability, flexibility, and reliability, NetArmor seamlessly integrates with any Netty based framework, ensuring your network is safeguarded against evolving threats. Its user-friendly interface simplifies management, making enterprise-grade security accessible to all.

## Notes
- **Continual Evolution**: Stay ahead with regular updates, as NetArmor grows every day.
- **Innovative AI-Based IDS**: Coming soon - an AI-driven intrusion detection system for enhanced security.
- **Join the NetArmor Movement**: Contribute and collaborate as we strive to set new benchmarks in Netty-based application security.
- **Code Quality & Contribution**: We adhere to stringent code quality standards. Join us in our commitment to security excellence.
- **Holistic Security Approach**: Remember, security is an ongoing journey. NetArmor is a vital component, but it’s part of a larger security strategy.
- **Community-Driven Development**: We believe in “Building Security Together”. Your feedback and contributions are invaluable.

## Features
- [x] Validators (Custom and implemented like email, credit card, etc.)
- [x] XSS, Path Transversal protection
- [x] Secure memory allocation and obfuscation for storing sensitive data in cache
- [x] Powerful password hashing algorithms implemented (Argon2id, BCrypt, SCrypt)
- [x] Security Policy Management (CSP, XFO, etc.)
- [x] HTTP/2 0day RST flood protection (see in [Cloudflare Blog](https://blog.cloudflare.com/zero-day-rapid-reset-http2-record-breaking-ddos-attack/))
- [x] HTTP Rate Limiting to prevent DoS, brute force attacks etc.
- [x] IP Whitelisting/Blacklisting
- [x] TLS (JA3) fingerprinting
- [x] HTTP/2 fingerprinting
- [x] Intrusion Detection System (IDS)

### Reactive components:
- Blacklisting
- Whitelisting
- Intrusion Detection System (IDS)
- TLS (JA3) Fingerprinting
- HTTP/2 Fingerprinting

Non-reactive components, like password hashing and validators, are optimized for utmost efficacy and security.
## Installation
First add exploit.org's maven repository:
### Maven:
```xml
<repositories>
    <repository>
        <id>exploit</id>
        <name>Exploit Repository</name>
        <url>https://maven.exploit.org</url>
    </repository>
</repositories>
```

### Gradle:
```groovy
repositories {
    mavenCentral()

    maven {
        url("https://maven.exploit.org")
    }
}
```
Then add NetArmor's dependency to project:
### Maven
```xml
<dependency>
    <groupId>org.exploit</groupId>
    <artifactId>netarmor</artifactId>
    <version>1.0</version>
</dependency>
```
### Gradle
```groovy
implementation 'org.exploit:netarmor:1.0'
```

# NetArmor Basic Features
First of all, you need to create a NetArmor instance.
```java
public class Main {
    public static void main(String[] args) {
        NetArmor armor = NetArmor.create();
    }
}
```

To specify custom your `SecurityConfiguration` instance,
or already implemented JSON based CommonSecurityConfig, use:

```java
public class Main {
    public static void main(String[] args) {
        SecurityConfiguration securityConfig; /* implement it */
        NetArmor armor = NetArmor.create(securityConfig);
    }
}
```
## Validators
Improper input validation is one of the most common security vulnerabilities, and it is the root cause of many other security vulnerabilities, including XSS, SQL Injection, etc.

NetArmor provides a set of rules, that can be used to validate user input,
with ability to customize them or create your own validators.

### Default rules:
- `username`
- `password`
- `email`
- `uuid`
- `phoneNumber`
- `url`
- `ipv4`
- `ipv6`
- `creditCard`
- `ssn`

Thanks to `ihateregex.io` for providing some of the regexes.

Please note, that as password should be strong enough, it is recommended to use `password` rule for password validation.
Rule specifies, that password should be at least 8 characters, 1 uppercase, 1 lowercase, 1 digit, 1 special character.

### Exception thrown:
- `ValidationException` - if input is invalid
- `UnknownRuleException` - if rule is not found.

Unknown rule exception is thrown, because it is better to throw an exception, than to allow the input to pass through the validation, and cause security vulnerability.
Every rule can be defined in `SecurityConfig` instance.

### Sample

```java
public class Main {
    public static void main(String[] args) {
        NetArmor armor = NetArmor.newArmor();

        var mail = armor.validator().input().validate("email", "my@mail.com");
    }
}
```
## HtmlEncoder
NetArmor provides a simple HTML encoder, that can be used to encode user input, before displaying it on the web page
and sanitize html from XSS attacks.

For sanitizing html, AntiSamy library by OWASP is used: [AntiSamy](https://owasp.org/www-project-antisamy/)

### Exception thrown
- `SanitizationException` - if input is invalid

### Sample
```java
public class Main {
    public static void main(String[] args) {
        NetArmor armor = getNetArmor(); // Implementation of NetArmor
        
        var encoded = armor.htmlEncoder().encode("<script>alert('XSS')</script>");
        var sanitizedHtml = armor.htmlEncoder().sanitizeHtml("your html");
    }
}
```

## Password Hashing
Strong password hashing is one of the most important security features, that every application should have, in order to protect user passwords from being stolen
or cracked by brute force attacks.

The first and foremost, password hashing algorithm should be slow, so that it takes a lot of time to hash a password,
never use MD5, SHA1, SHA256, SHA512, etc. for hashing passwords, because they are designed to be fast, and can be cracked easily.

NetArmor provides a set of password hashing algorithms, that can be used to hash user passwords.
By 2023, the most secure password algorithms are:
- `Argon2id`
- `SCrypt`
- `BCrypt`

Although BCrypt is not the most secure algorithm, it is still secure enough to be used in applications and is popular
mostly in legacy applications.
But it is recommended to use `Argon2id` or `SCrypt`, with <b>Argon2id</b> being the most secure algorithm
if properly configured.

All configurations are set to default values, but you can change them, if you want to make them more secure.
The default password encoder is specified in SecurityConfiguration instance.

### Sample
```java
public class Main {
    public static void main(String[] args) {
        NetArmor armor = getNetArmor(); // Implementation of NetArmor
        
        var password = "password";
        var hashedPassword = armor.password()
                .encode(password);
        
        var isPasswordValid = armor.password()
                .matches(password, hashedPassword);
    }
}
```

## File Path Transversal Protection
It is a common security vulnerability, that allows an attacker to access files, that are outside of the web root directory.
NetArmor provides a simple protection against this vulnerability, that can be used to protect your application.

### Exception thrown
- `SanitizationException` - if input is invalid or contains path transversal characters

### Sample
```java
public class Main {
    public static void main(String[] args) {
        NetArmor armor = getNetArmor(); // Implementation of NetArmor
        
        var sanitizedPath = armor.validator()
                .path()
                .validate("/etc/passwd");
    }
}
```

## Secure Memory Allocation
NetArmor provides a simple solution for storing sensitive data in memory, that is not accessible by other applications,
can not be swapped to disk, and is obfuscated.

Valid memory allocation is required for storing sensitive data, including passwords, tokens, etc, that are crucial for security of your application.
```
Please note, that only Linux and MacOS are supported for now.
```

### Sample
```java
public class Main {
    public static void main(String[] args) {
        NetArmor armor = getNetArmor(); // Implementation of NetArmor
        
        var secureMemory = armor.memory().allocate(11); // Allocates 11 bytes of secure memory
        secureMemory.write("Hello World".getBytes()); // Writes "Hello World" to secure memory
        secureMemory.obfuscate(); // Obfuscates secure memory
        
        var helloWorld = secureMemory.deobfuscate(bytes -> new String(bytes)); // Deobfuscates secure memory and returns "Hello World"
        // Memory is still obfuscated, if you want to deobfuscate it permanently, call secureMemory.deobfuscate()
    }
}
```

# Netty Security Handlers
As netty based servers may have different handlers in different order, you should first create NettyServer instance.

By the current date (2023-12-04) following servers are implemented via separate dependencies:
- `ReactorNettyProvider` - For reactor netty based servers.

To use it, please add following dependency:
### Maven
```xml
<dependency>
    <groupId>org.exploit</groupId>
    <artifactId>netarmor-reactor-netty</artifactId>
    <version>1.0.0</version>
</dependency>
```
### Gradle
```groovy
implementation 'org.exploit:netarmor-reactor-netty:1.0.0'
```

To create own instance of NettyServer, implement `NettyServer` interfaces:
```java
public interface NettyServerProvider<T> {
    void addFirst(ChannelPipeline pipeline, String name, ChannelHandler handler);

    void addBeforeHttpRequestHandler(ChannelPipeline pipeline, String name, ChannelHandler handler);

    void addBeforeHttp1RequestHandler(ChannelPipeline pipeline, String name, ChannelHandler handler);

    void addBeforeHttp2Handler(ChannelPipeline pipeline, String name, ChannelHandler handler);

    void addBeforeSslHandler(ChannelPipeline pipeline, String name, ChannelHandler handler);

    void addAfterHttpTrafficHandler(ChannelPipeline pipeline, String name, ChannelHandler handler);

    void addLast(ChannelPipeline pipeline, String name, ChannelHandler handler);

    NettyServerPipeline<T> newPipeline();
}
```

And pipeline itself:
```java
public interface NettyServerPipeline<T> {
    void addMitigationHandler(Supplier<MitigationHandler> mitigationHandler);

    T configure(T bootstrap);
}
```

## Example pipeline configuration

```java
public class Main {
    public static void main(String[] args) {
        SecurityConfiguration config = getSecurityConfig(); //e.g. method to get security config
        
        NetArmorPipeline<HttpServer> armor = NetArmorPipeline.newBuilder(new ReactorNettyProvider())
                .config(config) // if not specified default is used
                .intrusion(intrusionHandler)
                .whitelist(whitelistLimiter)
                .blacklist(blacklistLimiter)
                .tlsFingerprint(tlsPacketHandler)
                .http2Fingerprint(htt2PacketHandler)
                .build();
        
        // Now configure your server
        // As we specified ReactorNettyProvider, we will use HttpServer itself.
        // You can still implement your own provider, for you case.
        var server = HttpServer.create();
        armor.configure(server);
    }
}
```

To specify custom handlers, please extend `MitigationHandler` interface, or one of its implementations:
- **InboundMitigationHandler** - for inbound mitigation (`ChannelInboundHandlerAdapter` under the hood)
- **OutboundMitigationHandler** - for outbound mitigation (`ChannelOutboundHandlerAdapter` under the hood)

And add them to the builder:
```java
public class Main {
    public static void main(String[] args) {
        NetArmorPipeline<HttpServer> armor = NetArmorPipeline.newBuilder(new ReactorNettyProvider())
                .mitigation(mitigationHandler)
                .build();
    }
}
```

## Basic Security Handlers
### HTTP Flood Protection
HTTP Flood is a type of Distributed Denial of Service (DDoS) attack, that is used to make a web server unavailable by sending a large number of requests to the server.
Protection is enabled by default, but you can change the configuration in SecurityConfig instance.

#### Secure by default.
### HTTP/2 0day RST Flood Protection
HTTP/2 0day RST Flood is pretty new attack, that is based on sending a large number of RST frames and resetting the connection.
This type of protection is enabled by default, but you can change the configuration in SecurityConfig instance.

#### Secure by default.
### Security Policy Management
Security policy management is a set of rules, that are used to protect your application from XSS, Clickjacking, etc.
SecurityConfig instance contains default security policy management rules, but you can change them, if you want to make them more secure.

#### Secure by default.

## Enhanced Security Handlers
NetArmor provides a set of enhanced security features, that can be used to protect your application from various attacks,
including bot attacks, traffic from malicious/infected clients, etc.

### TLS (JA3) Fingerprinting
#### Reactive.

First of all, what is TLS fingerprinting? It is a technique, that is used to identify a client based on the TLS handshake,
exactly on the Client Hello message. It is a pretty new technique, that is used by many companies, including Cloudflare, to identify malicious clients.

As it is specific to every application, NetArmor provides a simple interface, that can be used to implement TLS fingerprinting:
```java
public class MyTlsFingerprinter implements FingerprintPacketHandler {
    @Override
    public Mono<ResultCode> handle(ChannelHandlerContext ctx, ClientHello ch) {
        var ja3 = ch.ja3();
        
        var hash = ja3.md5(); // md5 hash of ja3 fingerprint
        var raw = ja3.value(); // raw ja3 fingerprint
        
        // Do something with ja3
        // for example, check if it is in the blacklist
        // P.S isBlacklisted should be implemented by you
        
        if (isBlacklisted(ja3))
            return Mono.just(ResultCode.BLOCK);
        
        return Mono.just(ResultCode.OK);
    }
    
    @Override
    public String name() {
        return "my-tls-fingerprinter"; // Unique name used in the pipeline
    }
}
```
When ResultCode.BLOCK is returned, the connection is closed immediately,
there is no need to close it manually. (e.q calling `ctx.close()`)

Don't forget to add your fingerprint handler to NetArmor instance:
```
NetArmorPipeline.newBuilder(<your provider>)
        .tlsFingerprint(new MyTlsFingerprinter())
        .build();
```

#### Possible use cases:
- Allowing only specific clients to connect to your application (e.g. mobile clients)
- Blocking malicious clients (e.g. requests sent by malware on user's computer)
- Detecting bot attacks

### HTTP/2 Fingerprinting
#### Reactive.

HTTP/2 Fingerprinting is a technique, that is used to identify a client based on the HTTP/2 connection preface.
It is not very popular, but still can be used to identify if the client is browser or not.

To handle HTTP/2 fingerprinting, you need to implement `Http2FingerprintPacketHandler` interface:
```java
public class MyHttp2Fingerprinter implements Http2FingerprintPacketHandler {
    @Override
    public Mono<ResultCode> handle(ChannelHandlerContext ctx, Http2Settings settings) {
        // Do something with settings
        // for instance, you can check if settings don't correlate with the browsers' settings
        
        if (isSuspicious(settings))
            return Mono.just(ResultCode.BLOCK);
        
        return Mono.just(ResultCode.OK);
    }
    
    @Override
    public String name() {
        return "my-http2-fingerprinter"; // Unique name used in the pipeline
    }
}
```
When ResultCode.BLOCK is returned, the connection is closed immediately.
Don't forget to add your fingerprint handler to NetArmor instance:
```
NetArmorPipeline.newBuilder(<your provider>)
        .http2Fingerprint(new MyHttp2Fingerprinter())
        .build();
```

#### Possible use cases:

- Client Mismatch Detection
- Bot and Script Identification
- Targeted Attack Recognition

### IP Whitelisting/Blacklisting
#### Reactive.

NetArmor provides a simple interfaces, that can be used to implement IP whitelisting/blacklisting:

```java
public class MyWhitelist implements WhitelistLimiter {
    // Assuming that whitelist is stored in memory
    private final Set<String> whitelist = new HashSet<>();

    @Override
    public Mono<Boolean> isAllowed(String address) {
        return Mono.just(whitelist.contains(address));
    }
    
    @Override
    public String name() {
        return "my-whitelist"; // Unique name used in the pipeline
    }
}
```
```java
public class MyBlacklist implements BlacklistLimiter {
    // Assuming that blacklist is stored in memory
    private final Set<String> blacklist = new HashSet<>();

    @Override
    public Mono<Boolean> isBlocked(String address) {
        return Mono.just(blacklist.contains(address));
    }
    
    @Override
    public String name() {
        return "my-blacklist"; // Unique name used in the pipeline
    }
}
```

Don't forget to add your whitelist/blacklist handler to NetArmor instance:
```
NetArmorPileine.newBuilder(<your provider>)
        .whitelist(new MyWhitelist())
        .blacklist(new MyBlacklist())
        .build();
```

#### Possible use cases:
- Blacklisting malicious IP addresses
- Enabling access only for specific IP addresses

### Intrusion Detection System (IDS)
#### Reactive.

IDS is a set of rules, that are used to detect attacks, that are not detected by other security features.

It gives you ability to detect attacks, that are not detected by other security features, including 0day attacks,
or even identify new attacks, that are not known yet.

Although realization depends on the application, we already work on AI Based IDS, that will be available soon.
We plan to make it open source, so that everyone can contribute to it and make it better.

It will be advanced enough to detect attacks, that are not detected by other security features.

Information gathering consists of 4 steps:
- Remote Address gathering
- TLS (JA3) fingerprint gathering, if applicable (TLS must be used)
- HTTP/2 Settings gathering, if applicable (HTTP/2 must be used)
- HTTP Request gathering

All data in combination allows detect everything, starting from basic attacks based on path checking,
requests with strange data, ending with checking if data is spoofed, by trying to apply TLS fingerprint with
HttpRequest.

NOTE: TLS is available only if TLS is used, HTTP/2 is available only if HTTP/2 is used.

Example of implementation:
```java
public class MyIntrusionDetector implements IntrusionDetector {
    @Override
    public Mono<DetectionResult> detect(IntrusionDetectionData data) {
        var clientHello = data.getClientHello(); // Client hello packet, if TLS is used
        var ja3 = clientHello.ja3(); // Get Ja3 fingerprint
        
        var encoded = ja3.md5(); // md5 hash of ja3 fingerprint
        var raw = ja3.value(); // raw ja3 fingerprint
        
        var request = data.getRequest(); // The request that is being checked
        var ip = data.getRemoteAddress(); // IP address of the client
        
        var http2Settings = data.getHttp2Settings(); // HTTP/2 settings, if HTTP/2 is used
        // P.S you can collect data in case you want to train own model in future
        // or even to contribute the dataset to us.
        // Everything depends on privacy policy of your application, if it is allowed.
        
        // DetectCode can be OK, SUSPICIOUS or MALICIOUS
        return Mono.just(new DetectionResult(DetectCode.OK, data, "User is OK"));
    }

    // Will be called if the request is detected as malicious or suspicious
    @Override
    public Mono<HandleCode> onDetected(DetectionResult result) {
        // Do something with the result
        // for example, block the user or recheck the request
        // P.S you can use result.data() to get the data, that was passed to detect method
        
        // HandleCode can be PROCEED or BLOCK
        return Mono.just(HandleCode.BLOCK);
    }
    
    @Override
    public String name() {
        return "my-intrusion-handler"; // Unique name used in the pipeline
    }
}
```

Don't forget to add your intrusion handler to NetArmor instance:
```
NetArmorPipeline.newBuilder(<your provider>)
        .intrusion(new MyIntrusionDetector())
        .build();
```

### Sample use cases:
#### Detecting bots
Bot detection is a pretty hard task, because bots are getting smarter every day.
But comparing the data, that is provided by HTTP/2 settings, TLS Fingerprinting and
the data that bot tries to introduce in HTTP Request can be used to detect bots.

For instance, if client introduces itself as *well-known browser*, but sends HTTP/2 settings, that are not same with *well-known browser*,
or sends TLS fingerprint, properties of which are not used by or differs from *well-known browser*, it is most probably a bot.
Although TLS fingerprint spoof is possible, it is not very common, because it is pretty hard for realization.
Client can not simply intercept the packet and change data in it, because there are integrity checks in TLS.
###### What potentially bad user should do, we will not describe here, because it is not the topic of this article, moreover we don't want our security FAQ to be used by attackers to improve their attacks.

#### Detecting 0day attacks

Identifying 0day attacks is challenging due to their unknown nature and sophisticated tactics. However, by analyzing certain aspects of network traffic, it's possible to identify potential 0day threats.

- Monitor for unusual HttpRequest behaviors that deviate from the norm. This could include irregular request sequences, unexpected header values, or unusual request payloads. These anomalies might indicate an attempt to exploit unknown vulnerabilities.
- Compare the TLS fingerprints and Http2Settings of incoming traffic against known profiles of legitimate users. For example, if the TLS fingerprint or HTTP/2 settings don't match what's expected for the declared client type (like a well-known browser), this inconsistency can be a red flag.
- Look beyond the surface of HttpRequests to understand the intent. Sequential or repetitive requests targeting specific endpoints or unusual data patterns may indicate scanning activities often associated with 0day exploits.
- Track requests to identify common scanner patterns or source information that's typically associated with malicious activities. This includes scrutinizing the IP address for known bad actors or geolocations commonly linked to attackers.

#### ... and many more

Thus, IDS is a powerful tool, that can be used to detect attacks, that are not detected by other security features.

#### We remind again, that we are working on AI Based IDS, that will be available soon.

## Conclusion
NetArmor is a powerful security solution, that can be used to protect your application from various attacks,
but although it can be used as a complete security solution, it is recommended to use it as a part of your security solution.
There is no 100% security, it is a continuous process, that requires a lot of effort.

## Contributing

#### AI Based IDS
Please contact us by email: `contact@exploit.org` if you want to contribute to AI Based IDS.
More data we have, more accurate IDS will be.

#### Repository
Please make sure before you create a pull request, that your code is clean, readable, and maintainable.
Follow KISS, DRY, SOLID, YAGNI, and other principles.

### FOR ANY NON-PUBLIC SECURITY ISSUES, THAT YOU DON'T WANT TO DISCLOSE, PLEASE CONTACT US BY EMAIL:
### `contact@exploit.org`

```
In the industry, there may arise security concerns that you prefer not to disclose publicly but wish to address. 
We are committed to assisting you and collectively making the world more secure 
by swiftly implementing security features and patches for Java applications.
While we stay abreast of current security trends, identify new vulnerabilities, 
and implement innovative security measures, we believe that more eyes on a problem lead to better solutions.
We welcome your participation and collaboration in strengthening the security of IT systems.
```

## License
NetArmor is licensed under the BSD License. See [BSD](LICENSE) for more information.

 









