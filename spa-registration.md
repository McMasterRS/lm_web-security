---
layout: default
title: Microsoft Azure SPA Registration
parent: Authentication
nav_order: 1
---

# Microsoft Azure SPA Registration

Before adding SSO to your application, you will need to register it on the Microsoft Azure Directory by contacting UTS. Start a support ticket with UTS using the [Technology Services Jira Portal](https://macservicedesk.mcmaster.ca/plugins/servlet/desk/portal/742). You will need to provide them with a redirect URI. If your application is still in development, you can simply use the common name that you entered when creating the CSR file followed by a port number e.g., `dev-server.mcmaster.ca:8081`.

Once you SPA registration is complete, you will receive an email from UTS with the app registration information. The app information document will contain the `App-Id` also known as a `client_id`, a `tenant_id`, the `Secret Value` and `Secret Id`, in addition to the the application `Endpoints`. With this information in hand, we can start adding SSO to our SPA. 


