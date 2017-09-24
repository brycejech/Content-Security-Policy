# Content Security Policy (CSP)

## What is CSP?

Content Security Policy (CSP) is an added layer of security that helps to detect and mitigate certain types of attacks, including Cross Site Scripting (XSS) and data injection attacks. These attacks are used for everything from data theft to site defacement or distribution of malware.

[Content Security Policy on MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

A content security policy allows us to instruct the browser on the origins from which scripts, styles, images, frames, and other web resources are allowed to be loaded and executed for any given web page.

A CSP should not be your only layer of defense against XSS attacks. User input still should never be trusted and always be properly sanitized. However, when leveraged correctly, a robust content security policy can be an extremely versatile addition to your defense in depth strategy.

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

```nginx
add_header Content-Security-Policy "<your policy>";
```

#### Apache

In the `httpd.conf` file for the host or in an `.htaccess` file, add:

```apache
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

Popular web frameworks also provide interfaces through which you can set custom headers. However, if you plan to have a singular policy for your entire site, the recommended approach would be to add the `Content-Security-Policy` header directly via the web server.

If you do not have access to set HTTP response headers on your server, a CSP can also be specified with an HTML `<meta>` tag inside the document `<head>`:

```html
<meta http-equiv="Content-Security-Policy" content="<your policy>" />
```

## CSP Examples

A CSP header allows us to define a whitelist of approved sources from which any given resource type may be loaded, giving us very fine control over the origins from which web resources can be loaded (including our own).

A CSP applies to a single web page only, it can be unique for a single page or consistent across an entire site. CSPs can also be combined; in the case of multiple CSPs, browsers will attempt to honor all of them. Note that if multiple CSP headers are specified, the policy can only get more restrictive, not less.

Before we look at a comprehensive list of CSP keywords and directives, let's look at a few examples.

```yaml
# Restrict all content to the site's origin (excluding subdomains)
Content-Security-Policy: default-src 'self';
```

```yaml
# Restrict JavaScript to be loaded via https: from the current origin, allow inline script (unsafe)
# Restrict CSS to load via https: from the current origin, allow inline styles (unsafe)
# Restrict all other content to be loaded from the current origin
Content-Security-Policy: default-src 'self'; script-src https: 'self' 'unsafe-inline'; style-src https: 'self' 'unsafe-inline';
```

Note that inline script and styles (as well as the use of functions such as `eval()`) are considered unsafe. Try to avoid using inline styles and scripts unless absolutely necessary.

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

### Why is the use of inline JavaScript and CSS considered unsafe?

Browsers do not have any way to tell trusted from untrusted inline code; they cannot see the difference between your inline code and unsafe code from an attacker that may have been rendered into the page without being properly sanitized.

#### What makes styles or scripts 'inline'

Scripts are considered inline if they are a part of a `<script>` tag that is not loaded via the `src` attribute. `onclick`, `onhover`, `onload`, `onerror` and other similar HTML attributes that can contain JavaScript code are also considered inline.

HTML `style` attributes as well as `<style>` elements are considered inline style.

When a CSP is enabled and the `script-src 'unsafe-inline'` or `style-src 'unsafe-inline'` directives are not specified, inline JavaScript and CSS evaluation are blocked. However, an inline script tag may also be evaluated if it has a valid `nonce-` attribute value, or if it matches an already specified `sha256-` hash.

#### What about `'unsafe-eval'`?

The `script-src 'unsave-eval'` directive allows the use of text to JavaScript functions such as `eval(string)`, `setTimeout(string)`, and `new Function()`. They are also considered unsafe because untrusted user input could make it's way into them. They are blocked unless the `'unsafe-eval'` value is set.

When using `setTimeout()`, only the string evaluation is blocked, if you pass `setTimeout()` an inline function, it will still work as expected

## CSP Keywords and Directives

### Keywords

Keyword         | Meaning
--------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------
'none'          | Matches nothing, blocks use of the specified source
'self'          | Refers to the current origin (excluding subdomains)
'unsafe-inline' | Allows inline JS and CSS (unsafe)
'unsafe-eval'   | Allows text to be evaluated as JS (unsafe)
data:           | Allows loading resources via the data scheme such as a base 64 image src
https:          | Restrict resources to https:
`nonce-`        | script or style tags can be evaluated if their `nonce` attribute matches the header value `<script nonce="f8uz0jlZq41ljc">alert('Hello World!')</script>`
`sha256-`       | Allow a script or style to execute if it matches the specified sha256 hash.

### Directives

Directive                 | Meaning
------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
default-src               | Used as a fallback for unspecified directives
script-src                | Restricts origins from which scripts can be loaded
style-src                 | Restricts origins from which stylesheets may be loaded
img-src                   | Restricts origins from which images can be loaded
connect-src               | Restricts origins to which you can connect via XMLHTTPRequests (XHR), WebSockets, and EventSource
font-src                  | Restricts origins from which fonts may be loaded
form-action               | Restricts endpoints for form submissions
frame-ancestors           | Restricts origins from which the current page may be loaded in a frame element. Applies to `<frame>`, `<iframe>`, `<embed>`, and `<applet>`. Setting this to `'none'` is roughly equivalent to `X-Frame-Options: DENY`
object-src                | Allows control of Flash and other plugins
media-src                 | Restricts origins from which `<video>` and `<audio>` can be loaded
frame-src                 | DEPRECATED, use child-src
child-src                 | Restricts URLs for workers and frames. `child-src: https://youtube.com` would allow frames to be loaded from youtube.com, but not from any other origins.
base-uri                  | Restricts URLs that can appear in the `<base>` element
plugin-types              | Limits the types of plugins a page may invoke
report-uri                | URL to which the browser will POST policy violations
upgrade-insecure-requests | Instructs browsers to re-write URLs from http to https (useful for legacy sites with a lot of URLs that need to be re-written)


## Testing a policy with report-only mode

You may also supply a `Content-Security-Policy-Report-Only: <policy>` header, which will cause the browser to report on policy violations via the console as well as `POST` report violations to a `report-uri` (if specified) without blocking resources from loading or executing.

This feature can be used to help you fine-tune your policy. It can be used to test out new directives without making them part of the enforced policy, giving you time to address issues without breaking site functionality.

## Using the `report-uri` directive to collect policy violations

The `report-uri` directive is extremely useful for collecting information on report violations. There may be resources that we missed or an attacker actively trying to exploit an XSS attack vector. Aside from logged violations in the console, without policy violation reporting, we are left in the dark as to which directives may be being violated.

The `report-uri` directive is used to specify a URL to which the browser should `POST` a JSON formatted violation report in the event that it acts on the CSP.

Consider the following policy:

```yaml
Content-Security-Policy: default-src https: 'self'; report-uri https://cspreport.mydomain.com;
```

Let's say that on a page at `https://mydomain.com/index.html` that is subject to this policy we have the following `img` tag: `<img src="http://mydomain.com/favicon.ico" />`.

Upon detecting that this resource is in violation of the policy, the browser will `POST` a JSON formatted report similar to this:

```json
{
    "csp-report": {
        "document-uri": "https://mydomain.com/index.html",
        "referrer": "",
        "blocked-uri": "http://mydomain.com/favicon.ico",
        "violated-directive": "default-src https: 'self'",
        "original-policy": "default-src https: 'self'; report-uri https://cspreport.mydomain.com;"
    }
}
```

The endpoint at `https://cspreport.mydomain.com` should capture this data and log it for later use. You can then use the data to identify the pages and resources which are most frequently violating the policies (high traffic) and address them first, then move on to violations that are seeing less traffic.


## Browser Support

Header                                 | Firefox | Chrome | Safari | IE            | Edge
-------------------------------------- | ------- | ------ | ------ | ------------- | ---------------
Content-Security-Policy `v2`           | 31+     | 40+    | 10+    | -             | 15 build 15002+
Content-Security-Policy `v1`           | 23+     | 25+    | 7+     | -             | 12 build 10240+
X-Content-Security-Policy `deprecated` | 4+      | -      | -      | 10+ _limited_ | 12+ _limited_
X-Webkit-CSP `deprecated`              | -       | 14+    | 6+     | -             | -

[Test if your browser supports CSP](https://content-security-policy.com/browser-test/)
