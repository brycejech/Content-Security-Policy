# Content Security Policy (CSP)

## What is CSP?

Content Security Policy (CSP) is an added layer of security that helps to detect and mitigate certain types of attacks, including Cross Site Scripting (XSS) and data injection attacks. These attacks are used for everything from data theft to site defacement or distribution of malware.

[Content Security Policy on MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

A content security policy allows us to instruct the browser on the origins from which scripts, images, iframes, and other web resources are allowed to be loaded and executed for any given web page.

A CSP should not be your only layer of defense against XSS attacks. User input should still never be trusted and always be sanitized. However, when leveraged correctly, a robust content security policy can be an extremely versatile addition to your defense in depth strategy.

## What is Cross Site Scripting (XSS)?

Cross-site scripting (XSS) is a security exploit which allows an attacker to inject into a website malicious client-side code. This code is executed by the victim's browser and lets the attackers bypass access controls and impersonate users.

According to the Open Web Application Security Project (OWASP), XSS was the third most common Web app vulnerability in 2013.

[XSS on MDN](https://developer.mozilla.org/en-US/docs/Glossary/Cross-site_scripting)

[XSS on OWASP](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS))

## What are HTTP headers?

HTTP headers are part of an HTTP request or response, they convey additional information about the request/response. A header consists of it's case insensitive name followed by a colon, then it's value (without line breaks). Leading white space before the value is ignored.

[HTTP Headers on MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)

Some common HTTP request headers include `Host`, `User-Agent`, `Referrer`, `Authorization`, `Cookie`, and many others.

Some common HTTP response headers include `Server`, `Content-Type`, `Content-Length`, `Set-Cookie`, `Date`, `Expires`, and many others.

## Using the `Content-Security-Policy` Header

Currently, the most widely supported version of CSP is version 2. All of the keywords and directives mentioned here are supported in this CSP version (though browser support is sometimes limited; more on that later).

To use a CSP, we must configure our server to return the `Content-Security-Policy` HTTP header:

```yaml
Content-Security-Policy: <policy>
```

### Setting the `Content-Security-Policy` header in popular web servers

#### Nginx
In the `server{ }` block in the `nginx.conf` file, add the following:

```Nginx
add_header Content-Security-Policy "<your policy>";
```

#### Apache
In the `httpd.conf` file for the host or in an `.htaccess` file, add:
```Apache
Header set Content-Security-Policy "<your policy>"
```

#### IIS

In your `web.config` file, add the following to your `<system.webServer>` node:
```xml
<system.webServer>
    <httpProtocol>
        <customHeader>
            <add name="Content-Security-Policy" value="<your policy>" />
        </customHeader>
    </httpProtocol>
</system.webServer>
```

Popular web frameworks also provide interfaces through which you can set custom headers. However, if you plan to have a single policy for your entire site, the recommended approach would be to add the `Content-Security-Policy` header directly via the web server.

If you do not have access to set HTTP response headers on your server, a CSP can also be specified with an HTML `<meta>` tag:

```html
<meta http-equiv="Content-Security-Policy" content="<your policy>" />
```

## CSP Examples

Before we look at a comprehensive list of CSP keywords and directives, let's look at a few examples.

```yaml
# Restrict all content to the site's origin (excluding subdomains)
Content-Security-Policy: default-src 'self';
```

```yaml
# Restrict JS to https: via trusted.com (including subdomains)
# Restrict CSS to https: via trusted.com (including subdomains) or the current origin
# All other content restricted to https: via the current origin

Content-Security-Policy: default-src https: 'self'; script-src https: *.trusted.com; style-src https: 'self' *.trusted.com;
```

Note that values are not inherited from the `default-src` directive. Any directive completely overwrites the default for that type of resource. So, in the previous example, we had to re-declare `https:` for both the `style-src` and `script-src` directives, as well as re-declare `'self'` in the `style-src` directive. The `default-src` directive is used as a fallback for other directives that are not explicitly declared elsewhere.

```yaml
# Scripts may only be loaded via https from the /js/ folder on mydomain.com
# Styles may only be loaded via https from the /style/ folder on mydomain.com
# All other resources must be loaded via https from the current site's origin
Content-Security-Policy: default-src https: 'self'; script-src https://mydomain.com/js/; style-src https://mydomain.com/style/;
```

```yaml
# A more restrictive example.
# This would allow scripts, styles, and images to load from the
# current origin, and block everything else
Content-Security-Policy: default-src 'none'; script-src 'self'; style-src 'self'; img-src 'self'
```

## CSP Keywords and Directives



A CSP header allows us to define a whitelist of approved sources for a given resource, giving us very fine control over the origins from which web resources can be loaded (including our own).

A CSP applies to a single web page only, it can be unique for a single page or consistent across an entire site. CSPs can also be combined; in the case of multiple CSPs, browsers will attempt to honor all of them. Note that the policy can only get more restrictive, not less, if multiple CSP headers are specified.

There are a few keywords, such as `'self'`, that have special meaning.

Keyword         | Meaning
--------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------
'none'          | Matches nothing
'self'          | Refers to the current origin (excluding subdomains)
'unsafe-inline' | Allows inline JS and CSS (unsafe)
'unsafe-eval'   | Allows text to be evaluated as JS (unsafe)
data:           | Allows loading resources via the data scheme such as a base 64 image src
https:          | Restrict resources to https:
`nonce-`        | script or style tags can be evaluated if their `nonce` attribute matches the header value `<script nonce="f8uz0jlZq41ljc">alert('Hello World!')</script>`
`sha256-`       | Allow a script or style to execute if it matches the specified sha256 hash.

### Why is the use of inline JavaScript and CSS considered unsafe?

Browsers do not have any way to tell trusted from untrusted inline code; they cannot see the difference between your inline and code and unsafe code from an attacker that may have been rendered into the page without being properly sanitized.

Therefore, when a CSP is enabled and the `script-src 'unsafe-inline'` or `style-src 'unsafe-inline'` directives are not specified, inline JavaScript and CSS evaluation are blocked.

The `script-src 'unsave-eval'` directive allows the use of text to JavaScript functions such as `eval(string)`, `setTimeout(string)`, and `new Function()`. They are also considered unsafe and blocked unless this value is set.

`unsafe-eval` also applies to functions other than `eval()`; it also applies to `new Function()` and `setTimeout()` because strings can be passed to these functions and evaluated as JavaScript code. When using `setTimeout()`, only the string evaluation is blocked, if you pass `setTimeout()` an inline function, it will still work as expected

CSPs can include any of the following directives

Directive                 | Meaning
------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
base-uri                  | Restricts URLs that can appear in the `<base>` element
child-src                 | Restricts URLs for workers and frames. `child-src: https://youtube.com` would allow frames to be loaded from youtube.com, but not from any other origins.
connect-src               | Restricts origins to which you can connect via XMLHTTPRequests (XHR), WebSockets, and EventSource.
font-src                  | Restricts origins from which fonts may be loaded
form-action               | Restricts endpoints for form submissions
frame-ancestors           | Restricts origins from which the current page may be loaded in a frame element. Applies to `<frame>`, `<iframe>`, `<embed>`, and `<applet>`. Setting this to `'none'` is roughly equivalent to `X-Frame-Options: DENY`.
frame-src                 | DEPRECATED, use child-src.
img-src                   | Restricts origins from which images can be loaded.
media-src                 | Restricts origins from which `<video>` and `<audio>` can be loaded.
object-src                | Allows control of Flash and other plugins
plugin-types              | Limits the types of plugins a page may invoke
report-uri                | URL to which the browser will send CSP report violations
style-src                 | Restricts origins from which stylesheets may be loaded
upgrade-insecure-requests | Instructs browsers to re-write URLs from http to https (useful for legacy sites with a lot of URLs that need to be re-written).

## Leveraging CSP to Defend Against XSS Attacks

HTML `style` attributes as well as `<style>` tags are considered inline styles and will not be evaluated unless the `style-src: 'unsafe-inline'` directive is specified.

HTML `onclick`, `onhover`, `onload`, `onerror` and similar attributes as well as `<script>` tags are considered inline script and will not be evaluated unless the `script-src: 'unsafe-inline'` directive is specified.

## CSP Report-Only Mode

## Browser Support


Header                                 | Firefox | Chrome | Safari | IE            | Edge
-------------------------------------- | ------- | ------ | ------ | ------------- | ---------------
Content-Security-Policy `v2`           | 31+     | 40+    | 10+    | -             | 15 build 15002+
Content-Security-Policy `v1`           | 23+     | 25+    | 7+     | -             | 12 build 10240+
X-Content-Security-Policy `deprecated` | 4+      | -      | -      | 10+ _limited_ | 12+ _limited_
X-Webkit-CSP `deprecated`              | -       | 14+    | 6+     | -             | -

[Test if your browser supports CSP](https://content-security-policy.com/browser-test/)
