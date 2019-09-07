---
title: Generating   SSL/TLS certificate
layout: post
category: TLS
date: '2019-09-06 23:15:05'
---

Summary:

In this post, I am going to disucss about how to generate self-signed certificate and verify the certificate. At first, I will discuss different parts of SSL/TLS certificate and will show how to generate private key , certificate sigining request and  public key. Then, I will discuss how to discover the public key and CA. Finally, we will finish it by looking at PKI. 


Security is an complicated issue. To secure a system, we need focus on every single component of the platform. However, when transfering data from server to client and establishig a connection between server to server is chalenging. As we need to make sure the data is not tempared and no one in the middle can not evasdrop. TLS (Upgraded version of SSL) protocol desinged to ensure that. TLS and SSL term is used interchangabley.  In this post, I am not going to discss the TLS protocl but the certificate used by the protocol. 


Server implements the TLS certificate to transfer data over a secure channel.  This certificate is generated using PKI. The certificate consist of public key and private key. Lets deep dive how to generate this keys. In this demonastration, we are not going to use CA (Certificate Authority). CA plays the role of trusted party. 

### Introduction to  openssl 

openssl is a tool widely used as  to generate private key and public key pair. 

```
openssl version
```

### Generating private key

First setp in preparation for generating publik key is to generate private key. For key generation, we need to identify the algorithm, size and pharspras. The most commonly used key generation algorithm is RSA.  genrsa command can be used to generate the RSA key. It is recommend not to use the key size less than 2048 bit. 
```
openssl genrsa  -aes256 -out private.pem 2048
```

The command includes -aes256 flag to encrypt the RSA key. The command prompt to enter a passphrase to encrypt the key.

To look at the key structure, try following command. 

```
openssl rsa -text -in server.key
```

### Generating certificate sigining request

The next step is to generate public key. But the public key is generated through the certificate sigining request. The CSR is the formal request to ask the CA to generate a certificate which contains the public key. CSR is singed with the private key generated in the previous step. The information in the CSR will be used to generate the x509 certificate. The CSR command asks few question to collect information to place in the certificate. The most important part is the Common Name (Fully qualified host name).

```
openssl req -new -key private.pem -out public.csr
```

To inspect the CSR, use following command. 

```
openssl req -text -in server.csr -noout

```

### Generating public key
In this stage, we can pass the certificate sigining request (CSR ) to certificate authority to sing the public key.  However, in some cases, we do not need to use CA. In that cases, we can use the private key to sing the public key and generate x509 certificate.  In  other  word, private key is an untrusted CA that can be used to sing CSRs. The issue with self-signed certificate is that browser generates a warning that the certificate does not have a trusted thrid party. 

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
openssl req -x509 -newkey rsa:2048 -days 365 -keyout site1.pem -out site1.crt
```


### Connect to a domain


```
openssl s_client -connect tanverhasan.com:443
```

Finally, these are some of the `openssl` common commands. To read more about openssl, I woud recommed the following books. 
https://www.feistyduck.com/library/openssl-cookbook/online/ch-openssl.html