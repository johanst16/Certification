Skapa nycklar enligt nedan:

Detta gjordes på Linux Ubuntu 22 (HP Laptop)

https://localhost:8443/user


https://www.baeldung.com/x-509-authentication-in-spring-security

passwd : changeit

	SERVER CERTIFICATE

openssl req -x509 -sha256 -days 3650 -newkey rsa:4096 -keyout rootCA.key -out rootCA.crt

	Common Name (e.g. server FQDN or YOUR name) []:Baeldung.com	

openssl req -newkey rsa:2048 -keyout PRIVATEKEY.key -out MYCSR.csr

	Common Name (e.g. server FQDN or YOUR name) []:localhost


File : localhost.ext

	authorityKeyIdentifier=keyid,issuer
	basicConstraints=CA:FALSE
	subjectAltName = @alt_names
	[alt_names]
	DNS.1 = localhost

openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -in MYCSR.csr -out localhost.crt -days 365 -CAcreateserial -extfile localhost.ext


print content:

openssl x509 -in localhost.crt -text



openssl pkcs12 -export -out localhost.p12 -name "localhost" -inkey PRIVATEKEY.key -in localhost.crt

keytool -importkeystore -srckeystore localhost.p12 -srcstoretype PKCS12 -destkeystore keystore.jks -deststoretype JKS


*************************************************************************************************************************************


Root CA Installation
we need to install our generated root certificate authority as a trusted certificate in a browser.

An exemplary installation of our certificate authority for Mozilla Firefox would look like follows:

    1. Type about:preferences in the address bar
    2. Open Advanced -> Certificates -> View Certificates -> Authorities
    3. Click on Import
    5. Select the rootCA.crt file and click OK
    6. Choose “Trust this CA to identify websites” and click OK

Note: If you don't want to add our certificate authority to the list of trusted authorities, you'll later have the option to make an exception and show the website tough, even when it is mentioned as insecure. But then you'll see a ‘yellow exclamation mark' symbol in the address bar, indicating the insecure connection!


*****************************************************************************************************************************************

Mutual Authentication
client-side authentication

Truststore
It holds the certificates of the external entities that we trust.
In our case, it's enough to keep the root CA certificate in the truststore.
Let's see how to create a truststore.jks file and import the rootCA.crt using keytool:

keytool -import -trustcacerts -noprompt -alias ca -ext san=dns:localhost,ip:127.0.0.1 -file rootCA.crt -keystore truststore.jks


*******************************************************************************************************************************************

Client-side Certificate

First, we have to create a certificate signing request:

openssl req -new -newkey rsa:4096 -nodes -keyout clientBob.key -out clientBob.csr
Common Name (e.g. server FQDN or YOUR name) []:Bob


Next, we need to sign the request with our CA:

openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -in clientBob.csr -out clientBob.crt -days 365 -CAcreateserial

The last step we need to take is to package the signed certificate and the private key into the PKCS file:

openssl pkcs12 -export -out clientBob.p12 -name "clientBob" -inkey clientBob.key -in clientBob.crt

Finally, we're ready to install the client certificate in the browser.

Again, we'll use Firefox:

    1. Type about:preferences in the address bar
    2. Open Advanced -> View Certificates -> Your Certificates
    3. Click on Import
    5. Select the clientBob.p12 file and click OK
    6. Input the password for your certificate and click OK

*******************************************************************************************************************************


Create certificate for Alice

openssl req -new -newkey rsa:4096 -nodes -keyout clientAlice.key -out clientAlice.csr
openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -in clientAlice.csr -out clientAlice.crt -days 365 -CAcreateserial
openssl pkcs12 -export -out clientAlice.p12 -name "clientAlice" -inkey clientAlice.key -in clientAlice.crt