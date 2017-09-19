=======
title: Content Security Policy
=======

# Content Security Policy (CSP)

## What is CSP?

Content Security Policy (CSP) is an added layer of security that helps to detect and mitigate certain types of attacks, including Cross Site Scripting (XSS) and data injection attacks. These attacks are used for everything from data theft to site defacement or distribution of malware.

[Content Security Policy on MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

A content security policy allows us to instruct the browser on the origins that scripts, images, iframes, and other web resources are allowed to be loaded and interpreted for any given web page.

A CSP applies to a single web page only, it can be unique for a single page or consistent across an entire site. CSPs can also be combined; in the case of multiple CSPs, browsers will attempt to honor all of them.

A CSP gives us very fine control over the origins from which web resources can be loaded (including our own).

To use a CSP, we must configure our server to return the `Content-Security-Policy` HTTP header:

```yaml
Content-Security-Policy: <policy>
```

If you do not have access to set HTTP response headers on your server, a CSP can also be specified with an HTML `<meta>` tag:

```html
<meta http-equiv="Content-Security-Policy" content="<policy>" />
```

### What is Cross Site Scripting (XSS)?

Cross-site scripting (XSS) is a security exploit which allows an attacker to inject into a website malicious client-side code. This code is executed by the victim's browser and lets the attackers bypass access controls and impersonate users.

According to the Open Web Application Security Project (OWASP), XSS was the third most common Web app vulnerability in 2013.

[XSS on MDN](https://developer.mozilla.org/en-US/docs/Glossary/Cross-site_scripting)

#### XSS Example

A common example of an XSS attack is when an attacker is allowed to submit data to a server, typically as part of an HTTP POST or GET request, that is saved to a datastore and later interpreted (as HTML or JavaScript) on a webpage that other users (victims) will see. This is an example of a stored XSS Attack.

### What are HTTP headers?

HTTP headers are part of an HTTP request or response, they convey additional information about the request/response. A header consists of it's case insensitive name followed by a colon, then it's value (without line breaks). Leading white space before the value is ignored.

[HTTP Headers on MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)

Some common HTTP request headers include `Host`, `User-Agent`, `Referrer`, `Authorization`, `Cookie`, and many others.

Some common HTTP response headers include `Server`, `Content-Type`, `Content-Length`, `Set-Cookie`, `Date`, `Expires`, and many others.

## Leveraging CSP to Defend Against XSS Attacks

Currently, the most widely supported version of CSP is version 2. All of the keywords and directives mentioned here are supported in this CSP version (though browser support is sometimes limited, more on that later).

A CSP header allows us to define a whitelist of approved sources for a given resource.

There are a few keywords, such as `'self'`, that have special meaning.

Keyword         | Meaning
--------------- | ---------------------------------------------------
'none'          | Matches nothing
'self'          | Refers to the current origin (excluding subdomains)
'unsafe-inline' | Allows inline JS and CSS
'unsafe-eval'   | Allows text to be evaluated as JS

CSPs can include any of the following directives

Directive | Meaning
--------- | -------
base-uri | Restricts URLs that can appear in the `<base>` element
child-src | Restricts URLs for workers and frames. `child-src: https://youtube.com` would allow frames to be loaded from youtube.com, but not from any other origins.
connect-src | Restricts origins to which you can connect via XMLHTTPRequests (XHR), WebSockets, and EventSource.
font-src | Restricts origins from which fonts may be loaded
form-action | Restricts endpoints for form submissions
frame-ancestors | Restricts origins from which the current page may be loaded in a frame element. Applies to `<frame>`, `<iframe>`, `<embed>`, and `<applet>`.
frame-src | DEPRECATED, use child-src.
img-src | Restricts origins from which images can be loaded.
media-src | Restricts origins from which `<video>` and `<audio>` can be loaded.
object-src
plugin-types
report-uri
style-src
upgrade-insecure-requests

## CSP Report-Only Mode

## Browser Support
