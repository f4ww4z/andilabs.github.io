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
tl;dr *things are not what they appear*

On the lower layers of communication there are no hànzì chars or other chars beyond ASCII. **PUNYCODE**ing happens.
e.g `日本.com` is converted to `xn--wgv71a.com`

more on punycode:
[https://en.wikipedia.org/wiki/Punycode](https://en.wikipedia.org/wiki/Punycode)
[https://tools.ietf.org/html/rfc3492](https://tools.ietf.org/html/rfc3492)

web converter can be found here: 
[https://www.punycoder.com/](https://www.punycoder.com/)


risks:
======

![fake apple.com looking pretty legit](/assets/punycode-fake-apple-website.jpg)

demo: [https://www.xn--80ak6aa92e.com/ ](https://www.xn--80ak6aa92e.com/) will be converted to for user in web browser (till security update) as `https://www.apple.com` what more with green padlock standing for valid certificate!!! STRANGE

![seems legit](/assets/seems-legit.jpg)



webdeveloper issues:
====================

Be aware that `html5` form field `email` will pass all chars submited to your backend being already punycoded!! So be aware when implementing custom filtering mechanism to not search for raw stuff like `mason@日本.com` but rather `mason@xn--wgv71a.com`

python libs for conversion [https://pypi.python.org/pypi/idna](https://pypi.python.org/pypi/idna)
