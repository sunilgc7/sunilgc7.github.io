# Website security policies

All nine HTML pages declare a Content Security Policy (CSP) immediately after
the character encoding, before any resources load, and a
`strict-origin-when-cross-origin` referrer policy.

The CSP blocks scripts, embedded frames, plugins, network connections, base URL
overrides, and form submissions. It permits local images and styles, Google
Fonts stylesheets from `fonts.googleapis.com`, and font files from
`fonts.gstatic.com`. External links and email links continue to work. Update
the policy in every HTML page when intentionally introducing new resource types.

## GitHub Pages limits

This repository is served as static files on GitHub Pages. GitHub Pages does
not provide repository configuration for custom HTTP response headers.
Adding `_headers`, `.htaccess`, or entries in `_config.yml` will not configure
these headers on GitHub Pages.

The HTML policies provide browser protection but do not create HTTP response
headers. A scanner may still report missing headers. In particular,
`frame-ancestors` cannot be delivered in a CSP meta tag, and HSTS,
`X-Content-Type-Options`, and `X-Frame-Options` require actual response headers.
Blocking frames loaded by this site does not prevent other sites from framing it.

## Completing the header configuration

To address the remaining findings, use hosting with configurable response
headers, or a proxy in front of a custom domain. Moving hosting or changing DNS
is separate from these source changes. Configure the following on all responses,
including errors, at that hosting layer:

```http
Content-Security-Policy: default-src 'none'; style-src 'self' https://fonts.googleapis.com; font-src https://fonts.gstatic.com; img-src 'self'; base-uri 'none'; form-action 'none'; frame-ancestors 'none'; upgrade-insecure-requests
Referrer-Policy: strict-origin-when-cross-origin
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Strict-Transport-Security: max-age=300
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Resource-Policy: same-origin
```

Start HSTS with the short lifetime shown above. After confirming reliable HTTPS,
increase to `max-age=31536000`. Add `includeSubDomains` or request preload only
after verifying that every affected subdomain supports HTTPS permanently.

COEP is optional for this portfolio and is not required to fix the reported
failures. Enable it only after checking that all external resources, including
Google Fonts, work with the chosen isolation policy. No cookies or scripts are
currently used, so cookie attributes and script SRI do not need changes.

After deployment, check response headers with:

```sh
curl -I https://sunilgc7.github.io/
curl -I https://sunilgc7.github.io/hobbies.html
```

Use the actual deployed domain if hosting changes. Check the browser console on
every page for CSP violations, confirm typography and hobby images render, test
external links, and rerun the security scan. The source changes alone cannot
verify deployment or resolve the headers controlled by GitHub Pages.

References:

- [MDN: Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP)
- [MDN: frame-ancestors](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy/frame-ancestors)
- [MDN: Referrer-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Referrer-Policy)
