[![](https://jitpack.io/v/valb3r/letsencrypt-helper.svg)](https://jitpack.io/#valb3r/letsencrypt-helper)

# What is this

If you have ever tried getting Let's Encrypt certificate for Spring Boot application, you know that it is painful as
it involves using either CertBot or Docker-sidecar/Cron-job to manage certificate lifecycle. This library solves these
problems by managing certificate lifecycle directly in Java code with the help of awesome [Acme4j](https://github.com/shred/acme4j) library.

## Key features:

1. Obtain Let's Encrypt certificate on fresh start
2. Store generated keys and certificate into single KeyStore (`server.ssl.keystore`)
3. Renew Let's Encrypt certificate (it watches for certificate expiration date and updates it to new before old is expired)
4. **No JVM restart needed** when certificate gets updated

# Application requirements

To perform HTTP-01 ACME (Automatic Certificate Management Environment) challenge, the application must listen on port `80`, this library will automatically create
Tomcat connector to this port, so the only thing needed on your side is to open `80` port for the application.

# Servlet containers supported (embedded)

 - Tomcat
 
# Usage

## From JitPack maven repository

1. Import this library:
    1. [Add JitPack repository to your dependency management system](https://jitpack.io/)
    2. Add this library to your dependencies, [like it is shown here](https://jitpack.io/#valb3r/letsencrypt-helper/0.1.4.1)
2. Declare `@Import(TomcatWellKnownLetsEncryptChallengeEndpointConfig.class)` on your configuration
3. Define following properties in your application configuration or environment:
    1. `lets-encrypt-helper.domain` the domain to issue certificate for
    2. `lets-encrypt-helper.contact` your contact for Let's Encrypt (i.e. your email in format `mailto:john.doe@example.com`)
4. Configure SSL as usual for Tomcat+TLS using `server.ssl.keystore` for certificate and keys storage


## Configuration


| Property                                                   | Description                                                                                                  | Default value, if any  |
|------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------|------------------------|
| server.ssl.key-store                                       | Path to the KeyStore, where Let's Encrypt certificates and account key are to be stored (or are already there) |                        |
| server.ssl.key-store                                       | KeyStore type (i.e. PKCS12)                                                                                  |                        |
| server.ssl.key-store-pasword                               | Password for KeyStore with Let's Encrypt certificate and account key                                           |                        |
| server.ssl.key-alias                                       | Let's Encrypt certificate key alias in the keystore                                                            |                        |
| server.port                                                | Port (secure SSL/TLS) on which your application is deployed                                                  |                        |
| lets-encrypt-helper.domain                                 | Your applications' domain (i.e. example.com)                                                                 |                        |
| lets-encrypt-helper.contact                                | The contact of person responsible for the domain (i.e. mailto:john@example.com)                              |                        |
| lets-encrypt-helper.account-key-alias                      | Account key alias                                                                                            | letsencrypt-user       |
| lets-encrypt-helper.letsencrypt-server                     | Let's Encrypt server to use                                                                                    | acme://letsencrypt.org |
| lets-encrypt-helper.key-size                               | Certificate and Account key RSA key size                                                                     | 2048                   |
| lets-encrypt-helper.update-before-expiry                   | Start trying to update certficate this time before expiration                                                | P7D (7 days)           |
| lets-encrypt-helper.busy-wait-interval                     | Busy wait interval for thread that checks if the certificate is valid                                        | P1M (1 minute)         |
| lets-encrypt-helper.account-cert-validity                  | Validity duration for Account key                                                                            | P3650D (3650 days)     |
| lets-encrypt-helper.enabled                                | Is the helper enabled                                                                                        | true                   |
| lets-encrypt-helper.development-only.http01-challenge-port | For development only, port for HTTP-01 ACME challenge                                                        | 80                     |


### Example configuration

Launch your application with `-Dspring.profiles.active=ssl`

`application-ssl.yaml`:
```yaml
server:
  port: 443
  ssl:
    key-store: file:/home/user/letsencrypt/application-keystore # Path to KeyStore with certificates and keys
    key-store-password: change-me # Password for KeyStore protection
    key-store-type: PKCS12
    key-alias: tomcat # Certificate name in KeyStore
    enabled: true # Important to place this explicitly
lets-encrypt-helper:
  domain: my-domain.example.com # Domain to issue certificate for
  contact: mailto:john.doe@mymail.example.com # Your contact for Let's Encrypt
```

**Note:** On your server ensure you have opened port `80` for Java (i.e. in Firewall) and Java can bind to it (i.e. follow [Linux allow listening to low port without sudo](https://superuser.com/a/892391) to open ports 80,443 for `java`)

**Example project** with SSL and Let's Encrypt management using this library **[is located here](example)**

## Alternative

The library is itself just 1 Java class. You can add library dependencies and 
[this java file](https://github.com/valb3r/letsencrypt-helper/blob/master/src/main/java/com/github/valb3r/letsencrypthelper/TomcatWellKnownLetsEncryptChallengeEndpointConfig.java)
to your configiration
