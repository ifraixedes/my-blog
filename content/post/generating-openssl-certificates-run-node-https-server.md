+++
title = "Generating OpenSSL certificates to run a NodeJS HTTPS Server"
description = "Running a NodeJS HTTPS server is easy, however when you have to create the SSL certificates, it becomes a tedious task, so in this post I wrapped it up"
date = "2015-04-22T00:00:00+00:00"
tags = ["SSL", "HTTPS", "OpenSSL", "programming"]
categories = [
  "Software Development"
]

aliases = [
  "/post/generating-openssl-certificates-run-node-https-server",
  "/2015/04/22/generating-openssl-certificates-run-node-https-server/"
]
+++

The last few days I've had to implement a simple HTTP/HTTPS client wrapper for NodeJS, which returns a <a href="https://promisesaplus.com/" target="_blank">Promise</a>) for each request call and do some basic management for the common body structure that the APIs, which we're building, respond.

Nonetheless, I'm not writing this post for the itself implementation because the purpose doesn't matter to tell about the process to generate the SSL certificates to use for a iojs/NodeJS HTTPS server which you may need, as I needed, to test a HTTPS client.

As you may know and/or think, <a href="https://www.openssl.org" target="_blank">OpenSSL</a> isn't the most friendly library, or in this case command line tool, that you've ever been used, however I'm not complaining about it, it provides a useful functionality which, you may think the same than me, is not the most funny thing that you would like to implement, for that reason and because it's Open Source, I much appreciate the people how contribute to it, <a href="http://en.wikipedia.org/wiki/OpenSSL" target="_blank">as many others may do</a>

After the introduction, let's go for the interesting part of the post.


## Generating the OpenSSL certificates

To create a certificate which it's not self signed, to be able to emulate more how the real world works and avoid that the client report `self signed certificate` SSL/TLS error, first of all we have to have a <a href="http://en.wikipedia.org/wiki/Certificate_authority" target="_blank">CA (Certificate Authority)</a>.

The CAs' role is to emit certificates providing the trust (as a trusted third party) of the certificates that the serves use to proof that they're who they really are.

In the real world, the trusted CAs are regulated, in someway which guarantee that the world (a.k.a us) can trust on them due the security requirements that they fulfill; obviously for our test environment we don't need to know about all the bureaucracy.

Hence, first we have to __create the CA certificate__, in this case with one, the root, will be enough, so it must be self signed, because it's the root. In the real world the root CAs don't usually sign end certificates, they sign certificates for other CAs creating an hierarchy chain where the CAs from some certain bottom level sign the end certificates, the ones used by server/clients to prove their authenticity.

To generate it with OpenSSL, we just use <a href="https://www.openssl.org/docs/apps/req.html" target="_blank">`req` utility</a>

```
openssl req -batch -newkey rsa:2048 -nodes -keyout \
  root-ca.key -days 3650 -x509 -out root-ca.crt
```

Let's clarify some of the used options:

* `batch`: We avoid the interactive mode, so we execute and get.
* `nodes`: Specify not to encrypt the private key, so server can read it without any passphrase or key agent.
* `x509`: Generate a self signed certificate rather than a certificate request (we'll see this in a moment).


Now, we have our CA, then let's create the certificate that we need for our server; first we have to create a __certificate request__ with our private key; I'd just like to remind you that your private keys are generate by yourself and __never are sent anywhere__, which is the basis of a PKI (Public Key Infrastructure), share your public key and non-disclose never your private key.

```
openssl req -newkey rsa:2048 -batch -nodes \
  -config openssl.cnf -out localhost.csr -keyout localhost.key
```

Let's clarify some of the used options for this, as well

* `newkey`: Generate a new private key besides the certificate request; if we'd already had one, we'd remove this option and will provide another one to specify the file which contains the key, but with this, we save the call to another command.
* `config`: It provides a OpenSSL configuration file where we can specify what values and default values; it's important here because I want to specify `localhost` as domain name (in an Open SSL certificate is the "common name") to match with the host used in the test's requests but at the same time I want to assign in automatically without having to reply interactive questions (`-batch`)

The configuration file, in this case `openssl.cnf`, for the purpose that I want, only needs two fields, in the format that OpenSSL expect (if I'm not wrong it accepts other ones, but I know exactly this); it looks like this

```
[ req ]
distinguished_name     = req_distinguished_name

[ req_distinguished_name ]
 commonName                      = Common Name (eg, your name or your server's hostname)
 commonName_default              = localhost
```

And finally, let's give the certificate request to the CA to generate the certificate for our server through <a href="https://www.openssl.org/docs/apps/x509.html" target="_blank">`x509` utility</a>

```
openssl x509 -req -CAkey root-ca.key -CA root-ca.crt \
  -CAcreateserial -in localhost.csr -days 3650 -out localhost.crt
```

And the options clarification

* `req`: Specify that the input is a certificate request than a certificate
* `CAcreateserial`: Generate a serial number file if doesn't exist, so if we run this for first time, it will, otherwise will read from it the serial number and increment it for the next time.


And we're done with OpenSSL; in my case I add this in a `makefile target` not to have to remember all of this each time that I have to generate them.


## Running a NodeJS HTTPS server

Running a HTTPS server with <a href="https://iojs.org/api/https.html#https_https_createserver_options_requestlistener" taget="_blank">iojs</a> / <a href="https://nodejs.org/api/https.html#https_https_createserver_options_requestlistener" target="_blank">NodeJS</a> is pretty straightforward as a HTTP one.

I could save this section however, because the API documentation example is quite clear however, I thought that it doesn't hurt to add here to provide a better understanding to the post.

{{< highlight js >}}
var https = require('https');
var fs = require('fs');
var path = require('path');

var options = {
  key: fs.readFileSync(path.join(__dirname, 'localhost.key')),
  cert: fs.readFileSync(path(__dirname, 'localhost.crt'))
};

https.createServer(options, function (req, res) {
  res.writeHead(200);
  res.end("hello world\n");
}).listen(8000);
{{< /highlight >}}


## Requesting the server from NodeJS HTTPS client

Requesting the server from HTTPS <a href="https://iojs.org/api/https.html#https_https_request_options_callback" target="_blank">iojs</a> / <a href="https://nodejs.org/api/https.html#https_https_request_options_callback" target="_blank">NodeJS</a> client is pretty straightforward as well, however I have to provide the CA which signed the certificate used by the server, otherwise client will report an error saying that the certificate is wrong, basically because it cannot trust in a certificate which has been emitted for an unknown CA.

Therefore, on each request we must specify the `ca` property in the options object with the CA's certificate, maybe it's annoying, but better than adding the certificate to the OS keychain, due that the <a href="https://nodejs.org/api/https.html#https_https_globalagent" target="_blank">`globalAgen`</a> ignores the <a href="https://nodejs.org/api/tls.html#tls_tls_connect_options_callback" target="_blank">`tls.connect`</a> options.

{{< highlight js >}}
var https = require('https');
var fs = require('fs');
var path = require('path');

var options = {
  hostname: 'localhost',
  port: 8000,
  path: '/',
  method: 'GET',
  ca: fs.readFileSync(path(__dirname, 'root-ca.crt'))
};

var req = https.request(options, function(res) {
  console.log("statusCode: ", res.statusCode);
  console.log("headers: ", res.headers);

  res.on('data', function(d) {
    process.stdout.write(d);
  });
});
req.end();

req.on('error', function(e) {
  console.error(e);
});
{{< /highlight >}}

And done!

Another solution to avoid to pass the CA certificate to the client is to use the `rejectUnauthorized` options property to false, but in that case you are authorising everything, so I prefer to add the used CA and limit the authorisation for non-official CAs to only the create one.


## Conclusion

Generating our certificate for our development/test environment with OpenSSL is a little bit tedious if you have to figure out how, however when you have how, it can be added to your "task runner" of your project and forget about the process.

On the other hand, running a HTTPS server and making HTTPS request with iojs/NodeJS are pretty straightforward, knowing a bit about the server and client requirements, because when you don't, finding out what happens with the error messages that OpenSSL sometimes gives, isn't a nice task.


I've hope that you've found this useful.
