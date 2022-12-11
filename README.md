A small demo showing how to create a valid digital signature for use with the eBay developer API using Python. It is intended to provide a starting point for anyone looking to add digital signature support into their own application. A gist containing only the main functions can be found here: https://gist.github.com/carlansell94/f1fecf94da53f5e51c065df5ed0492fb

By adding credentials to ```credentials.json```, you can run this code directly for debugging purposes. To do this, you'll need to have a signing key, generated using the eBay Key Management API. If you haven't already created a signing key, check the section 'Creating a Signing Key' below. The developer account used must have access to the ```sell.finances``` scope.

This demo uses an ED25519 key, though it should be simple to switch this out for RSA as required.

The following Python libraries are required:
* requests
* pycryptodome

Note that this will not work as-is with the eBay provided docker container. The container only works with the eBay provided test credentials, so you're better off testing with your actual credentials through the API proper.

## Overview
This section contains a brief overview of the requirements for a digital signature to be created successfully. This hopefully clears up some of the key points outlined in the IETF draft specification eBay have chosen to rely on, without providing any further explanation.

### Creating a Digital Signature
The digital signature has a very specific structure, with newlines (\n) required after each element.

The private key is imported using the ```pycryptodome``` library. The key needs to be in the following format (including line breaks):

```
-----BEGIN PRIVATE KEY-----
Base64 key...
-----END PRIVATE KEY-----
```

This format will be familiar to anyone that has created cryptographic keys in the past. If you only store the raw private key, be sure to add the 'BEGIN' and 'END' sections in code as required, and don't forget the line breaks! (This is how the demo code works).

The resulting signature needs to be base64 encoded.


### Creating a Signing Key
If you don't already have a key, you can run ```demo.py``` with the ```--generate-key``` argument to create one. Details of the created key will be printed, for you to make a copy of.

A new signing key can be created on your account at any time, and the API provides functionality to retrieve the public elements of previously-generated keys. This does *not* return the private key, which you must store a copy of yourself.


### Sending a POST Request
For POST requests, a content digest header containing a SHA-256 hash of the request body must be present. The digest should be base64 encoded, and the header value should take the form ```sha-256=:digest:```.

This app inclides a function to create a valid content digest. The content being encoded should be in string form, for example, ```'{"hello": "world"}'```.

The digital signature created for a POST request must include a "Content-Digest" field, with the calculated digest as the value. The signature input string also needs to include "Content-Digest".
