---
title: "App Transport Security"
layout: post
date: 2018-11-18
tags: iOS
---

App Transport Security (ATS) is a technology that requires an app to either support best practice HTTPS security or statically declare its security limitations via a property in its `Info.plist`.
It improves privacy and data integrity by ensuring your app’s network connections employ only industry-standard protocols and ciphers without known weaknesses.

Lets see how to handle the more common scenarios:
(The XML can be seen selecting the `Info.plist` file and then `Open As -> Source Code`)

## No changes to App Transport Security configuration.

Your HTTPS connection is [defined](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW33) as "secure" if all of the following match:
- Server must support at least Transport Layer Security `TLSv1.2`.
- The negotiated TLS connection cipher suite must support perfect forward secrecy (PFS).
- The leaf server certificate must be signed either with a (RSA) key with a length of at least 2048 bits or a (ECC) key with a size of at least 256 bits.

You can diagnose ATS connection issues using the `nscurl` tool.

``` bash
/usr/bin/nscurl --ats-diagnostics [--verbose] URL
```

## Ignore all App Transport Security Restrictions

When we are prototyping an app quickly or we don't want to handle this ATS thing yet we can bypass all restrictions by setting this configuration.

``` xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
```

In this template you can use any of the following, using preferably the most specific one depending on your needs.

- NSAllowsArbitraryLoads
- NSAllowsArbitraryLoadsForMedia
- NSAllowsArbitraryLoadsInWebContent
- NSExceptionAllowsInsecureHTTPLoads
- NSExceptionMinimumTLSVersion

For example if your app have an embedded web browser you may need to use `NSAllowsArbitraryLoadsInWebContent`.

Keep in mind that if you allow everything like in the example below and submit to the store, your app most likely will be rejected unless you provide justifications eligible for consideration.

## Add per-domain exceptions

This is the most common approach, you provide all the specific "non-secure" domains and put rules related to it.

``` xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>httpbin.org</key>
        <dict>
            <key>NSIncludesSubdomains</key>
            <false/>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <false/>
            <key>NSExceptionRequiresForwardSecrecy</key>
            <true/>
            <key>NSExceptionMinimumTLSVersion</key>
            <string>TLSv1.1</string>
            <false/>
        </dict>
    </dict>
</dict>
```

The following are the most common rules that you can apply.

- **NSIncludesSubdomains:** **(Boolean)** If set to YES applies the ATS exception to every subdomain of the specified domain.
- **NSExceptionAllowsInsecureHTTPLoads:** **(Boolean)** If set to YES this key allows us to use insecure http or with incorrect certificates. This key may trigger App Store review clarifications.
- **NSExceptionMinimumTLSVersion:** **(String)** If included you must specify the minimum TLS version required. Valid values are `TLSv1.0`, `TLSv1.1` and `TLSv1.2` (default).
- **NSExceptionRequiresForwardSecrecy:** **(Boolean)** If set to NO this key allows to use a cipher that doesn't support forward secrecy.

## References:
- [App Transport Security Official Doc](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW33)
- [Working with Apple application transport security](http://www.neglectedpotential.com/2015/06/working-with-apples-application-transport-security/)
- [App Transport Security at Use Your Loaf blog](https://useyourloaf.com/blog/app-transport-security/)
