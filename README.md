Opt-into Better Defaults
========================

mkwst@; Dec. 2022

A Problem
---------

There are many, many configuration options that we would like web developers to consider when setting up the frontend of their web applications. Cross-origin isolation, framing, injection mitigation, permission policy, the list is long.

Today, developers need to look at each option individually, because the defaults are generally terrible. If developers don't know about an option or simply forget to configure it, they're putting their users at risk (because, again, the defaults are terrible).

While we have a long-term goal of changing those defaults, perhaps there's something we can do in the short term to help developers along.

A Proposal
----------

Let's give developers a single header that changes all the defaults to things we think they should be using. They can then work from a solid baseline, relaxing constraints intentionally when it's reasonable to do so, given their application's threat model and framework.

That is, developers could send the following header with navigation responses:

```http
Baseline: Security=2022
```

If the developer did nothing else, it would act as though the developer had specified:

```js
Content-Security-Policy: script-src 'self';
                         object-src 'none';
                         base-uri 'none';
                         require-trusted-types-for 'script';
                         trusted-types 'none';
Cross-Origin-Embedder-Policy: credentialless
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Resource-Policy: same-origin
Permissions-Policy: /* TBD; some reasonable baseline configuration */
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
```

This would cross-origin isolate the page, prevent it from being embedded in other contexts, prevent it from being sniffed as anything other than its declared content-type, configure its capabilities in some reasonable way, prevent any cross-origin script from executing, and cut off APIs that allow DOM injection.

If the developer specifies any of these headers explicitly, that declaration would override the default entirely. For instance, if the developer really needed script from a set of sources beyond their own origin, they could specify them (or, better, we can steer them towards nonces and hashes).

_**TODO(mkwst)**: We might want to do something different for `Permissions-Policy`, insofar as it would be nice to just set all the defaults to something else (e.g. [window named item access](https://github.com/w3c/webappsec-permissions-policy/issues/349)), and require developers to override each. That might be the behavior we get through the union of these headers, rather than replacement?_

The important point is that they'd be encouraged to do so explicitly, rather than needing to know about the thing they need to change ahead of time. This would require substantially improved error messages for this state (e.g. "Hey, `innerHTML` assignment didn't work because you opted into sane defaults. Consider using `.setHTML()` instead, and if you just can't use that API, go read [this article on Trusted Types] and [this article on Strict CSP] and configure both by sending an explicit `Content-Security-Policy` header."

Also, note the "2022" in the header: presumably we'll invent better defaults tomorrow. We'd roll those into a 2023 update next December. And do the same on an ongoing basis forever (perhaps with some "`Please-Break-My-Site-Without-Warning`" that adopts the current best practice on a rolling basis).

FAQ
---

### Why is `Security` a dictionary key?

I'm imaginging that if this was a successful model for pulling a page's security
properties into modernity, it might plausibly be useful for other areas of web
technology that developers might not want to opt-into in lockstep. It seems
reasonable to me to support some extension in the future without adding new
headers (e.g. `Baseline: Security=2024, Layout=2022`).

### This is still developer opt-in. Could we use something like this to pull folks along?

_**TODO(mkwst)**: Talk about alternatives, such as John Wilander's "linked-on-or-after" notion, which would tie the year's features to the year's restrictions. That seems bigger and substantially harder to ship, but potentially more impactful._
