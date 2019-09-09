---
title: Generating   SSL/TLS certificate
layout: post
category: TLS
date: '2019-09-06 23:15:05'
comments: true
---

### Summary:

In this post, I am going to discuss how to generate a self-signed certificate and verify the certificate. At first, I will discuss different parts of SSL/TLS certificate and will show how to generate a private key, certificate signing request and public key. Then, I will discuss how to discover the public key and CA. Finally, we will finish it by looking at PKI. 


Security is a complicated issue. To secure a system, we need to focus on every single component of the platform. However, when transferring data from server to client and establishing a connection between server to server is challenging. As we need to make sure the data is not tempered and no one in the middle can not eavesdrop. TLS (Upgraded version of SSL) protocol designed to ensure that. TLS and SSL term are used interchangeably.  In this post, I am not going to discuss the TLS protocol but the certificate used by the protocol. 


The server implements the TLS certificate to transfer data over a secure channel.  This certificate is generated using PKC (Public Key Cryptography). RSA is the most widely used public-key encryption algorithm. It is an asymmetric encryption algorithm.   Lets deep dive on how to generate these keys. In this demonstration, we are not going to use CA (Certificate Authority). CA plays the role of a trusted party. 





### Introduction to  openssl 

openssl is a tool widely used as  to generate private key and public key pair. 

```
openssl version
```

### Generating private key

The first step in preparation for generating the public key is to generate the private key. For key generation, we need to identify the algorithm, size and pass phrase. The most commonly used key generation algorithm is RSA.  `genrs`a command can be used to generate the RSA key. It is recommended not to use the key size less than 2048 bit. 

```
openssl genrsa  -aes256 -out private.pem 2048
```

The command includes -aes256 flag to encrypt the RSA key. The command prompt to enter a pass phrase to encrypt the key.

To look at the key structure, try following command. 

```
openssl rsa -text -in server.key
```

### Generating certificate sigining request

The next step is to generate a public key. But the public key is generated through the certificate signing request. The CSR is the formal request to ask the CA to generate a certificate which contains the public key. CSR is singed with the private key generated in the previous step. The information in the CSR will be used to generate the x509 certificate. The CSR command asks a few questions to collect information to place in the certificate. The most important part is the Common Name (Fully qualified hostname).

```
openssl req -new -key private.pem -out public.csr
```

To inspect the CSR, use following command. 

```
openssl req -text -in server.csr -noout

```

### Generating public key

In this stage, we can pass the certificate signing request (CSR ) to the certificate authority to sing the public key.  However, in some cases, we do not need to use CA. In those cases, we can use the private key to sing the public key and generate an x509 certificate.  In another word, the private key is an untrusted CA that can be used to sing CSRs. The issue with a self-signed certificate is that the browser generates a warning that the certificate does not have a trusted third party. 

```
openssl x509 -req -sha512 -in public.csr -signkey private.pem -out public.pem
```

Inspect the whole certificate

```
openssl x509 -noout -text -in public.pem

```

To printe specifc information form certificate, grep command can be appended if you are linix or mac system. 

```
openssl x509 -noout -text -in server-cert.pem | grep Issuer

```

How do we verfiy a certificate is self-signed certificate? To answer this questin, look at the Issuer and Subject property in the certificate. If both Issuer and Subject has the same value, then the certificate is a self-signed certificate. 


### Simplify the command 

In the previous section, we generated private key and certficate sigining requst in two seperae command. But it is possible to generate both private key and certificate sigining key in one command. 

```
openssl req -newkey rsa:2048 -days 365 -out test.csr -keyout test.pem

```

If there is no need for CA, then openssl can be used to generate self-signed certificate and private key. The following command creates rsa 2048 bit key and self singed certificate from scratch. 

```
openssl x509 -in site1-certificate.pem -pubkey -noout > site1-public-key.pem
```

### Retrieving public key from x509 certificate. 

Many times authorized receviers just publish the certificate. It is possible to retrieve the public key from certificate and encrypt the data needs to be transmited securely. Then, recevier decrypt the data using their private key. The folloiwong command can be used to obtain the public key from certificate. 

```
openssl x509 -in site-certificate.pem -pubkey -noout > site1-public-key.pem
```


### Connect to a domain


```
openssl s_client -connect tanverhasan.com:443
```

Finally, these are some of the `openssl` common commands. To read more about openssl, I woud recommed the following books. 
https://www.feistyduck.com/library/openssl-cookbook/online/ch-openssl.html


### Optional

Asymmetric encryption is computationally expensive. To make things easier, the most system combines both asymmetric and symmetric algorithm. For example, the bulk amount of data are encrypted using the symmetric algorithm. In the symmetric algorithm, the same key should be used to encrypt and decrypt data. Once the data is encrypted and transmitted to the receiver. The receiver needs the key used to decrypt the data. To transport the key, the asymmetric key algorithm is used.