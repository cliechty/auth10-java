This library speaks the WS-Federation protocol and SAML 1.1 and 2.0 tokens. It interops fine with Microsoft-related products like ADFS, Windows Azure Active Directory and Windows Identity Foundation.

The code is a fork of https://github.com/auth10-java/auth10-java.git but has been modified to also decrypt `EncryptedAssertion`'s. The master auth10-java github project is a simplified version with some improvements of the library released by Microsoft <https://github.com/WindowsAzure/azure-sdk-for-java-samples>. 

## Usage

Clone it

	git clone https://github.com/amsa-code/auth10-java.git

Or download it as zip from <https://github.com/amsa-code/auth10-java/zipball/master>

Import the Maven that was just downloaded in your project (File -> Import -> Existing Maven Projects)

Add a reference to `com.auth10.federation` library from your project. Edit your project Maven file `pom.xml` and add this:

```xml
<dependencies>
	...
	<dependency>
		<groupId>com.auth10.federation</groupId>
		<artifactId>auth10-federation</artifactId>
		<version>0.9-SNAPSHOT</version>
	</dependency>
	...
</dependencies>
```

## Using the Sample Project
Included in this repo is a sample webapp called 'wsf-sample'. To use this webapp you will need to customize the following files:
* `/src/main/resources/federation.properties`
* `/src/main/resources/rsa_private_key.pk8` (only required if using SAML2 encrypted assertions)

## How it works
The `/src/main/webapp/WEB-INF/web.xml` is configured so visiting anything after the base url of the webapp (/wsf-sample/*) will apply `WSFederationFilter.java` filter. This filter checks if the session is valid and if it's not valid it will redirect to the Identity Provider's authentication url. Once authenticated it returns back to the specified return url in `federation.properties`. The response that comes back will contain the claims under the **Assertion** element node in the SAML xml. If using SAML2 **and encryption** you'll find the claims under the **EncryptedAssertion**. This of course leads onto the next topic....


## Decrypting Assertions

If your Identity Provider(IP) uses SAML version 2 with encrypted assertions you will need to 
*replace the private key `/src/main/resources/rsa_private_key.pk8` in the wsf-sample project.
*install Java Cryptographic Extensions (JCE) for your JRE

JCE for Java 7 can be found here : http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html
Follow the readme found in the JCE zip.

If you generated your public/private key as a Java keystore (*.jks) then you can do the following to extract the private key.

First identify the alias name of the private key
```
keytool -list -v -keystore your-keystore-file.jks | grep "PrivateKeyEntry"
examplealias, 22/01/2010, PrivateKeyEntry, 
```

Extract the key into a PKCS12 container.
```
keytool -v -importkeystore -srckeystore you-keystore-file.jks -srcalias examplealias -destkeystore myp12file.p12 -deststoretype PCS12
```

Convert it to a PEM and then to a PKCS8 key.
```
openssl pkcs12 -in myp12file.p12 -nocerts -out key.pem
openssl pkcs8 -topk8 -nocrypt -inform PEM -in key.pem -outform DER -out rsa_private_key.pk8
```

You should have a resultant rsa_private_key.pk8 file that you can copy into your project's /src/main/resources/rsa_private_key.pk8

If the public/private key is *.pfx key generated by Microsoft Certificate Authority just do the following:
```
openssl pkcs12 -in mscertauth.pfx -nocerts -out key.pem
openssl pkcs8 -topk8 -nocrypt -inform PEM -in key.pem -outform DER -out rsa_private_key.pk8
```



## Consuming user attributes
In the sample webapp attributes are read in by the `ClaimServlet.java` and written to session output.

```java
// gets the user name
String name = request.getRemoteUser();

// gets the user claims
List<Claim> claims = ((FederatedPrincipal)request.getUserPrincipal()).getClaims()
```
