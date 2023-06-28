---
layout: default
title: Outline
nav_order: 3
---
## Goal of this Learning module

The goal of this learning module is to learn how to improve the security of a Next.js single-page application by using SSL certificates, enabling single sign-on using Microsoft Azure, using HTTP security headers and secure local storage. 

### Learning Objectives

At the end of this learning module, attendees will be able to:

- Request SSL certificates from UTS.
- Use the SSL certificates a dockerized Next.js application with Nginx
- Register your application in the Azure Active Directory.
- Add the Microsoft Authentication Library for React (`msal-react`) to your application.
- Create custom sign-in and sign-out MUI buttons for your webpage.
- Configure SSO in your SPA using the authentication flow.
- Use the redirect and pop-up workflows for SSO.
- Read and display the account information of the user who is currently logged in.
- Use six different HTTP security headers to protect against common security attacks.
- Encrypt data and store it securely in the browser's local storage.

## Table of Content
1. [SSL Certificates](certificates.md)
2. [Authentication](authentication.md)
	1. [SPA Registration](spa-registration.md)
	2. [Single Sign-On](sso.md)
	3. [Testing](testing.md)
3. [Security Headers](security-headers.md)
4. [Secure Local Storage](local-storage.md)
5. [Conclusion](conclusion.md)