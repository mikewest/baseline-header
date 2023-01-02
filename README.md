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

### `Baseline` only changes defaults. Could we use something like this to facilitate actual deprecations?

[John Wilander][wilander] has [floated][tpac2022] the notion of applying macOS/iOS SDK's
"[linked on or after][looa]" concept to the web, which is somewhat conceptually similar
to this document's proposal. My understanding of the proposal is something like the
following:

*   New APIs would be tied to some versioning scheme, whether time-based like this
    document's proposal, or versioned in some more esoteric manner akin to operating
    system versions. `form.requestSubmit()` might be exposed with version 4, while CSS
    Subgrid might not exist until version 8. 

*   Likewise, behavior deprecations would have a lifetime tied to a version. For example,
    `document.domain` might stop working at version 5, and synchronous XHR at version 8.

*   Developers would opt-into a particular version in some way, possibly by delivering
    a response header. This opt-in would both enable exciting new capabilities, _and_
    trigger the corresponding deprecations. With the examples above, a page that
    declared itself as being version 6 would have access to `form.requestSubmit()`,
    but not to `document.domain`.

Unlike the `Baseline` proposal above, this isn't merely about changing the defaults, but
about moving from deprecation to _removal_. New functionality would pull developers towards
newer versions, creating incentives to lock themselves out of deprecated behavior, thereby
providing a clear evolutionary path for the platform.

My feeling is that this idea is worth exploring, but is also substantially more complicated
than the `Baseline` proposal in this document. Socially, it's complicated to get folks to
agree that _their_ new thing ought to be tied to unrelated deprecations (see the general
inapplicability of `SecureContext` restrictions to CSS features, for example). Technically,
it's complicated to safely carry around multiple versions of behavior for a given API.

I'm also not sure what the end-game is for older versions. Ideally, we'd be able to simply
stop supporting them, as iOS or macOS can. However, the web's model is quite different,
and it's somewhat unexpected that a website relying upon deprecated behavior would stop
working when you upgrade your laptop. Eventual removal seems to have the same long-tail
challenges that we face with any other deprecation today. That said, this model might
well make a deprecation's impact a little more clear by giving developers a standardized
way of testing the implications of moving past any given milestone, and give browser
vendors more information about the set of sites actually relying on any particular
behavior, as that reliance will be somewhat more explicitly visible in datastes like
HTTP Archive or CrUX.

I'd like to start with changes to the platform's defaults, as that seems both tractable
and concrete. If we can evolve that mechanism into one which also productively ties the
carrots of new APIs to the sticks of deprecation/removal
(`Baseline: Security=2024, Platform=2022`?), so much the better.

[wilander]: https://hackerfiction.net/
[tpac2022]: https://github.com/w3c/webappsec/blob/main/meetings/2022/2022-09-12-TPAC-minutes-1.md#:~:text=JohnW%3A%20I%27ve%20brought%20this%20up%20before%2C%20but%20something%20like%20on%20OS%20linking%3A%20%22on%20or%20after%20XXX%22%2C%20you%20opt%20in%20to%20the%20new%20thing%20but%20can%27t%20use%20the%20old%20thing%20any%20more.
[looa]: https://developer.apple.com/documentation/uikit/uiapplication/1622952-canopenurl#discussion:~:text=If%20you%20link%20your%20app%20on%20or%20after%20iOS%209.0
