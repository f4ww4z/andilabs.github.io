---
layout: post
title:  "Punycode. Intro and risks."
date:   2017-04-28 10:31:17
categories: punycode security unicode beyond-ascii blog andilabs RFC3490 RFC3492
---


RFC 3490
========
tl;dr domain names can now include chars beyond ASCII. Legit examples:

`mason@日本.com`

`wildwezyr@fahrvergnügen.net`


punycode
========
tl;dr *things are not what they appear*; On the lower layers of communications there are no hànzì chars. **PUNYCODE**ing is being used.
e.g `日本.com` will be converted to `xn--wgv71a.com`

more on punycode: 
https://en.wikipedia.org/wiki/Punycode
https://tools.ietf.org/html/rfc3492

web converter can be found here: 
https://www.punycoder.com/


risks:
======

demo: https://www.xn--80ak6aa92e.com/ will be converted to for user in web browser (till security update) as https://www.apple.com what more with green padlock standing for valid certificate!!! STRANGE



