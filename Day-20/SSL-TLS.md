SSL/TLS:
========


This is mainly used for the secure comunication b/w the client and the server over the internet.

* SSL is the old method of communication.
* now a days we are using only TLS latest encryption.

* All the request in the HTTP formate only.



Case: 1 simple take ssh connection b/w the servers:
---------------------------------------------
          want to connect to 
server 1 -----------------------> server 2

* In this case we have to create pub.key and private.key in the server with the help of ssh-keygen.
* step-1 we have to copy the pub.key in to the server-2 .ssh/auth file.
* step-2 at the time of connect estrablesh we have to pass the private.key so in this case private key will go through internet. this is not a good
  practice if hacer steal the private key he will see the ecrypted data.

Case: 2 real flow b/w the client and server:
------------------------------------------
          HTTPS Connection
Clinet -------------------- > server 

* in this case we are creating a public.key, certificates, Private.key at server end with the help of openssl.
* step-1: when user initiate the request to server the server will provide s.public.key to client.
* clinet save this s.public.key and by using this s.pub.key it will encrypt the client.symantric.key.
* Step-2: this encrypted client.symentric.key will forword to server.
* Step-3: server private.key will decrypt the client.symentry.key.
* in this case hacer get the client.symentry key encrypted formate there is no use.
* in thise case also we have draw back what if insted of server we connected to hacker website and do all these things it will go wrost.


Case: 3 Certiface comes in to role:
-----------------------------------

* To over come case:2 issue we have certificates.
* The client browser will verify the server sertificate before exchanging the keys to server.
* Is the server certificate is signed by a proper CA organization or not.
* By using the certificate public key we will encrypt the client.symantry.key and send to server.


1. How server will get the certificate ?
A.server will get by CSR(Certificate sighing requet)

* this request is signed by CA organization before sign the certificate this CA will check the requested company, website and send the        certificate to server with public and private key.
* private key will stored in server.
* public key will sent to user browser.

Note:
-----
This above scenarios is mainly used fro public domines which are available over the internet.


Now what about for internally organized domines:
-------------------------------------------------


* for that we actually use a custom CA that are hosted with in our organization to sign the custome certificates.
* this is responsible fro issueing and signing the certificates.
