---
layout: default
title: Where to Store Data on the Client Side?
nav_order: 7
---

# Where to Store Data on the Client Side?

![local-session-cookies](assets/img/local-session-cookies.png)
_Image retrieved from [loginradius.com](https://www.loginradius.com/blog/engineering/guest-post/local-storage-vs-session-storage-vs-cookies/)_  

Web applications often need to store user data to help manage sessions and retain information when navigation from one page to another, performing requests or reload a page. One of the main disadvantage of the HTTP protocol is the "stateless" nature of the relationship between the browser and server. In other words, the HTTP protocol does not require the server to retain information or status about each user for the duration of multiple requests. To get around this limitation, web developers tend to store session or user information on the client side using the following methods: cookies, local storage, and session storage. In this section of the learning module, we will compare these 3 methods in terms of characteristics and use cases.

## Local Storage

Local storage is a read-only interface property that provides access to the website’s local storage object. Data stored in the local storage is saved across browser sessions. Local storage allows developers to store *key: value* pairs in the browser and is often considered an alternative to cookies. Data stored in the local storage is never transferred to the server.

**Persistence:** Local storage has no expiration time, i.e., it will not be deleted unless the user manually removes or the developers programs their web application to delete individual local storage items or clear the local storage after performing a particular action.

**Size:** Local storage provides 5 MB of storage space per domain, which is significantly more than the storage space offered by cookies.

**Type of data:** Local storage is limited to string *key: value* pairs. If the developer wants to store JSON data, then they will need to convert it to string first. JavaScript and TypeScript provide built-in methods to convert a JSON object to a string or vice versa:

```ts
//convert object to string
JSON.stringify(object_to_be_stored)    
//convert string back to object 
JSON.parse(string_value_from_local_storage)
```

**Security:** Local storage is vulnerable to Cross-site scripting (XSS) attacks, but not to Cross-Site Request Forgery (CSRF) attacks. An XSS attack happens when an attacker gains access to the data stored in a website's `localStorage` by running malicious scripts. Data stored in the local storage is saved in plain text, which means that local storage is not secure by design. Local storage is only accessible within the same domain. The main intent of storing data in local storage is accessing it in JavaScript, so there is no property to prevent local storage from being accessed using scripts. As such, developers should refrain from storing sensitive or personal session or user information in local storage, particularly if it is not encrypted.
## Session Storage

Session storage is a similar mechanism to local storage and allows developers to store data in the browser in the same *key: value* pair format. Data stored in the session storage is also never transferred to the server. 

**Persistence:** Unlike local storage, session storage persists only for the duration of the session i.e., it will  be deleted when the users closes their tab or browser. A new session is created each time a tab or window is opened. In other words, each tab/window that is opened with the same URL creates its own session storage instance. However, should the user duplicate a tab, then the session storage of the original tab will be copied to the duplicated tab. Closing a window/tab ends the session and clears all session storage objects.

**Size:** Session storage also provides 5 MB of storage space per domain. 

**Type of data:** Session storage shares the same type limitations as local storage, and as such is limited to string *key: value* pairs. JSON objects still need to be "stringified" before being stored in the session storage and parsed upon retrieval.

**Security:** Session storage shares the same vulnerabilities as local storage, i.e., it is vulnerable to XSS attacks, but is safe from CSRF attacks. Session storage is only accessible within the same domain, and there is no attribute to prevent session storage from being accessed using malicious scripts. Akin to local storage, developers should refrain from storing sensitive data in the session storage.
## Cookies

An HTTP cookie is a string, that like local and session storage is stored in the browser. Cookies are created and sent by the web server to the client. On the client side, the browser stores these cookies as string *key: value* pairs that can be sent to the server on subsequent requests. Data stored in cookies can be used to maintain the client’s state or session/user information for the stateless HTTP communication protocol. For instance, cookies are often used to store session tokens that can be used to verify that a user is logged in to a web application. Developers can also create cookies from the client side if needed.

**Persistence:** Cookies come in two types:
- **Session Cookies**: These cookies persist until the browser session ends, i.e., until the user closes their browser tab or window.
- **Permanent Cookies**: These cookies expire at pre-defined date and time specified by the developer using the `Expires` attribute.

**Size:** Cookies cannot store as much data as the local or session storage and are limited to 4 KB per domain.

**Type of data:** Cookies also store information in *key: value* pairs (e.g., `key1=value1; key2=value2;` ). Notice that a cookie may contain multiple *key: value* pairs, so the developer may have to parse the string to retrieve a specific key-value pair.

**Security:** Cookies are vulnerable to XSS and CSRF attacks, but they provide ways to mitigate such attacks:
- **HttpOnly**: The `HttpOnly` cookie header prevents the cookie from being accessed using JavaScript/TypeScript. Note that cookies created on the client side cannot be `HttpOnly`, which means they will remain insecure and vulnerable to XSS attacks. The `HttpOnly` cookie header can help mitigate XSS attacks, but is not sufficient on its own and should be combined with other properties (like `sameSite`) to improve the security of the web application.
- **SameSite:** The `sameSite` cookie attribute specifies rules on whether/when cookies are sent with cross-site requests. The `sameSite` header should be set to `Strict` to limit the cookie to HTTP requests to the same site where it originated. Setting this property to `Lax` can make your website vulnerable to CSRF and XSS attacks since the cookie will also be sent if a request to the website originates from another site. You can also combine the `sameSite` cookie header with anti-CSRF tokens to further improve the resilience of your website against CSRF attacks.
- **Secure:** When the `secure` flag of a cookie is set to true, the cookie may only be transmitted using a secure connection (SSL/HTTPS). This measure prevents cookies from being observed by unauthorized parties due to the transmission of the cookies in plain text.
