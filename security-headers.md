---
layout: default
title: Security Headers
nav_order: 6
---

# Securing the SPA Using HTTP Headers

An HTTP (HyperText Transfer Protocol) header is used to pass extra information with HTTP requests and responses. Each header is a key-value pair separated by a colon. 
When a user visits a website, the server receives a request from the browser. The server then responds to the browser and the response will include HTTP headers.

Headers give more context to a webpage and allow developers to protect their websites from various security attacks. Security headers provide the browser with more information regarding the resources that the website can access and how it behaves in relation to other websites that the user may be browsing simultaneously. 

By adding security headers to the response of our pages, we can minimize the possibility of the following attacks:

- Cross-Site Scripting (XXS): a web security vulnerability that allows an attacker to compromise the security of a web application by manipulating the website to force it to run harmful JavaScript code that enables the attacker to masquerade as the user and perform fraudulent activities on their behalf. 
- Clickjacking: an attack that tricks a user into clicking a webpage element which is invisible or disguised as another element. Clickjacking allows a hacker to force the user to download malware or provide credentials and sensitive information without their consent.
- Code injection: an attack that involves injecting malicious code into an application from untrusted sources. Websites that connect to a multitude of domains without any restrictions are particularly susceptible to this type of attacks.

Next.js allows you to set security headers from the `next.config.js` file situated in the main folder of your project. We will make use of six different HTTP security headers to help protect our application. 

#### `X-Frame-Options` header

The `X-Frame-Options` header is designed to prevent clickjacking attempts on a website by ensuring that the site’s content is not embedded in other websites.  

We will set the value of the `X-Frame-Options` header to `DENY` in order to prevent browsers from loading our website in an `<iframe>`, `<frame>`, `<object>`, or `<embed>` element. Attackers can use these elements to add invisible controls on top of the UI elements of our website to trick users into giving out information or unwillingly performing harmful tasks.  

#### `Content-Security-Policy` header

The `Content-Security-Policy` header allows you to set a policy for which domains are approved for executable scripts as well as the kind of resources (e.g., fonts, images, etc.) that can be loaded into your website.  

We will use the `Content-Security-Policy` header to allow resources to be loaded only from the same origin (`self`) and enable inline scripts and styles, which are necessary for Material UI (MUI) components. We will also authorize our website to connect to the `login.microsoftonline.com` domain for SSO in addition to accessing Google Fonts to download the Roboto family of fonts.  

#### `X-Content-Type-Options` header

The `X-Content-Type-Options` header is designed to disable Multipurpose Internet Mail Extensions (MIME) type sniffing, a technique used by browsers to determine type of a resource based on the response content instead of what is specified in the `Content-Type` header.  

Attackers can manipulate the MIME sniffing algorithm to force the browser into interpreting data in a way that allows them to carry out malicious operations like cross-site scripting.  

Using the the `nosniff` directive in the `X-Content-Type-Options` header forces the browser to adhere to the MIME types specified in `Content-Type` and thus reduces the possibility of XSS attacks.  

#### `Permissions-Policy` header

The `Permissions-Policy` header allows you to specify the Web APIs are allowed to be used within your website. This security header enables you to opt out of using external devices that are not needed for your website to function. The geolocation, camera, microphone APIs and many more should always be disabled if they are not used in your web application. Disabling unused web APIs helps lower the risk of attackers exploiting these channels for fraudulent or malicious activities.  

We will disable the camera, battery, geolocation and microphone web APIs since our demo website does not need them.  

#### `Referrer-Policy` header

If your website contains clickable links to another domain then , then your website is said to be a referrer.  Whenever a user clicks on one of those links, certain information about the referrer is sent to the target domain in the HTTP request’s referrer header.  

The `Referrer-Policy` header allows developers to specify how much information about the referrer is sent with the referrer header in each HTTP request when navigating from one domain to another.  

We will make use of  the `origin-when-cross-origin` directive for the `Referrer-Policy` to send the path, origin, and query string when performing a same-origin request between equal protocol levels. An example of an equal protocol level would be from HTTPS to HTTPS.  

#### `Strict-Transport-Security` header

The `Strict-Transport-Security` header instructs web browsers to connect to your website only via HTTPS , thus ensuring that every connection is encrypted and secure from infiltration by third parties. You can also specify the duration in seconds that the browser will remember this protocol, with the recommended duration being 31536000s or 1 year. We will also use the `includeSubDomains` directive to ensure that all subdomains also adhere tot eh HTTPS protocol requirement.  

### Adding HTTPS Security Headers to a Next.js SPA

To add HTTPS security headers to your Next.js SPA, simply open the  `next.config.js` file located in the root directory of your project and modify it as shown below:  

```ts
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  env: {
    CLIENT_ID: process.env.CLIENT_ID,
    TENANT_ID: process.env.TENANT_ID,
    REDIRECT_URI: process.env.REDIRECT_URI,
  },
  webpack: (config) => {
    config.module.rules.push({
        test: /\.md$/,
        use: 'raw-loader',
    })

    return config
  },
  async rewrites() {
    return [
        {
            source: '/api/:path*',
            destination: `${process.env.HOST}/api/:path*`,
        }
    ];
  },
    // Adding policies:
    async headers() {
        return [
            {
                source: '/(.*)',
                headers: [
                    {
                        key: 'X-Frame-Options',
                        value: 'DENY',
                    },
                    {
                        key: 'Content-Security-Policy',
                        value: "default-src 'self' https://login.microsoftonline.com; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com  https://fonts.bunny.net; font-src 'self' https://fonts.gstatic.com  https://fonts.bunny.net data:; img-src 'self' data:;",
                    },
                    {
                        key: 'X-Content-Type-Options',
                        value: 'nosniff',
                    },
                    {
                        key: 'Permissions-Policy',
                        value: "camera=(); battery=(); geolocation=(); microphone=()",
                    },
                    {
                        key: 'Referrer-Policy',
                        value: 'origin-when-cross-origin',
                    },
                    {
                        key: 'Strict-Transport-Security',
                        value: 'max-age=31536000; includeSubDomains',
                    }
                ],
            },
        ];
    },
}

module.exports = nextConfig
```
