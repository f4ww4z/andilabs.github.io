---
layout: post
published: true
title:  "Get fast translation by Google Translate of any text selected anywhere on your Mac OS X"
date:   2015-02-01 02:05:00
categories: utility translation google automator os-x mac-os-x 
---


How it works:
-------------

Very similar to Mac OS X build in feature `Search in Google`. You select whatever text you wish in any application on your Mac, and from context menu you choose  e.g `-> Translet me this by Google Translate` (you can call it whatever you wish and find intuitive), and the selected text gets passed to Google Chrome with url set to GoogleTranslate and given Text, so as the result you should see opened webpage with the result.

How it is done:
---------------

We use apple Automator (default Mac OS X tool) you can find in Applications.

After selecting `New` choose `Service`

![automator-service](/assets/automator_translate.png)


Drag from left pane `Run Apple Script` action and copy paste below script.

Cmd + S for save, and choose your desired name for service and enjoy its usage! Since now it should be avaliable in context menu evry time you select some text.



The script:
{% highlight applescript %}
-- Automator Service for passing selected text (after selection the service from context menu) to google translate (works in Google Chrome, OS X 10.10 still up-to-date)
on run {input}
	
	set inputText to input as string
	
	
	tell application "Google Chrome"
		activate
		set theUrl to "http://translate.google.com/#en/pl/" & inputText
		
		
		if (count every window) = 0 then
			make new window
		end if
		
		set found to false
		set theTabIndex to -1
		repeat with theWindow in every window
			set theTabIndex to 0
			repeat with theTab in every tab of theWindow
				set theTabIndex to theTabIndex + 1
				if theTab's URL = theUrl then
					set found to true
					exit repeat
				end if
			end repeat
			
			if found then
				exit repeat
			end if
		end repeat
		
		if found then
			tell theTab to reload
			set theWindow's active tab index to theTabIndex
			set index of theWindow to 1
		else
			tell window 1 to make new tab with properties {URL:theUrl}
		end if
	end tell
	
end run
{% endhighlight %}



