---
layout: default
title: SSL Certificates
nav_order: 4
---

# SSL Certificates

Before registering your SPA on the Microsoft Azure Directory, you will need to request SSL certificates from UTS. This section of the workshop will guide you through the process of generating the required files to request SSL certificates and setting up those certificates on your dockerized Next.js application. 

### Create CSR with OpenSSL

A Certificate Signing Request, CSR, is a file you obtain to request an SSL certificate from a Certificate Authority.

To generate the CSR with OpenSSL, you must first generate a private RSA key using the following command:
```
openssl> genrsa -des3 -out server.key 2048
```

The key should be at least a 2048-bit RSA key and must be encrypted using either Triple-DES or any other applicable method and must be stored in a PEM format so that it is readable as ASCII text.

Once the private key is ready, a Certificate Signing Request (CSR) can be generated using the following command:
```
openssl> req -new -key server.key -out server.csr
```

You will be prompted to enter your organization information and one critical attribute, common name. The common name is the fully qualified domain name (FQDN) of your server.

### Create a Support Ticket With UTS
Once you have the CSR file ready, you can start a support ticket with UTS using the [Technology Services Jira Portal](https://macservicedesk.mcmaster.ca/plugins/servlet/desk/portal/742). Click on "Submit an IT Request" and fill out the form. Make sure to mention that you are requesting a certificate for your website and that you already generated the CSR file. Remember to attach the CSR file to the ticket before submitting it. 

### Using the SSL Certificates
When your SSL certificates are ready, you will receive an email from Sectigo with links to download these certificates in various formats. We recommend using the "Certificate (w/ issuer after), PEM encoded" format. 

TODO
