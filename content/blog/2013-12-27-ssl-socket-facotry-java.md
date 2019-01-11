+++
title="Trusting a Self-signed SSL Certificate"
[extra]
tags="java, android, SSL, SSLException, self-signed"
+++

When connecting to a server which has setup their SSL encryption with a self-signed certificate
through a Java or an android app, you'll get an SSLException since their certificate isn't signed
by a Certificate Authorities (CA) which is a 3rd party who is trusted by everyone.

```text
javax.net.ssl.SSLException: Not trusted server certificate exception
```

If you have access to the server or know the site admin, get them to buy a CA signed SSL certificate.

But in case that is not possible or feasible we can one thing we can do is
configure our HTTP client library to just not verify the site hostname with the certificate.
But then again that is not a good solution and defeats the whole purpose of encryption.

<!-- more -->

A more elegant solution is to tell the client library to explicitly trust a certificate.
We'll have to manually get a copy of the site's self-signed certificate that we can use
to tell our client to trust.

Java has a couple of HTTP clients, mainly:
* Apache HTTP client
* HttpUrlConnection

You'll find that when using Apache Http client we'll have to create a custom SSLSocketFactory to implement the above 
functionality.
And for HttpUrlConnection set a custom SSLSocket to the connection as described at [developer.android.com][1]. 

Here is a reference implementation that does just that.

{{ gist(url="https://gist.github.com/shalzz/8125772") }}

#### For HttpURLConnection

```java
MySSLSocketFactory sslf = null;
try {
    KeyStore ks = MySSLSocketFactory.getKeystoreOfCA(getResources().openRawResource(R.raw.myCert));
    sslf = new MySSLSocketFactory(ks);
} catch (Exception e) {
    e.printStackTrace();
} finally {
    sslf.fixHttpsURLConnection();
}
```

Here we create a `Keystore` containing our certificate, in this case `myCert`.
Which we then use to get an instance of `MySSLSocketFactory` and then call its non static `fixHttpsURLConnection()` method.
This sets the SSLSocketFactory created as the default SSLSocketFactory for HttpURLConnection. 


#### For Apache Http Client

```java
MySSLSocketFactory sslf = null;
DefaultHttpClient client  = null ;
try {
    KeyStore ks = MySSLSocketFactory.getKeystoreOfCA(this.getResources().openRawResource(R.raw.myCert));
    client  = MySSLSocketFactory.getNewHttpClient(ks);
} catch (Exception e) {
    e.printStackTrace();
}
```

In this case we get a `DefaultHttpClient` created with the `KeyStore` containing our Certificates.

> Originally posted at [coding-euphoria.blogspot.in][2]  
> Edited and updated on 20/07/2017

[1]: https://developer.android.com/training/articles/security-ssl.html
[2]: https://coding-euphoria.blogspot.in/2013/12/custom-sslsocketfactory-that-trusts.html
