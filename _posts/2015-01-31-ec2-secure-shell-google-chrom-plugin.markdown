---
layout: post
published: true
title:  "Connect to EC2 via SECURE SHELL plugin for Google Chrome"
date:   2015-01-31 12:05:00
categories: ec2 amazon secure-shell
---


Befor you start you need:
-------------------------

![secure shell - plugin for Google Chrom](/assets/secure-shell-google-chrome.png)

* your private key dogspot.pem (dogspot is an example, you will have something else probabbly)
* installed secure shell plugin

Then you have to generate a public key from your private key. But before you have to change permissions to downloaded *.pem file, which really should be kept save and secret!

	chmod 400 dogspot.pem


To do this open your Terminal and run following command:

	ssh-keygen -y -f dogspot.pem > dogspot.pub

IMPORTANT: After this step you have to remove extension in file name dogspot.pem, so just dogspot left.

Then you should fill the fields somehow like shown below in the picture. First field can be descriptive name which is intuitive for you. The username by default is ubuntu and hostname you can take from your AWS Console under Public DNS and now the most important part.

Click Import... and select two files (the private key without pem !!) at once and the public key (with pub extension). If everything goes ok, you should see the name in the dropdown and be able to connect to your instance.

![select both files at once](/assets/blured.png)