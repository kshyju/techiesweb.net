---
layout: post
title:  "Rounded corner buttons in XCode"
date:   2015-07-12 12:11:33
categories: IOS
---

I have been dabbling with XCode and Swift for the past few days and thought of sharing some tips and tricks i learned.

###How to create a rounded corner button

There is no direct way to set corner radius for your button in xcode attribute inspector. So what you should do is, create a user defined run time attribute for corner radius.


Select your button, Go to **Identity Inspector**, there you will see a section called User Defined Runtime Attributes. Click on the **+** button to add a new record and enter the following values


	Key Path : layer.cornerRadius
	Type     : Number
	Value    : 8 //or whatever value you want for your corner radius.


![Settings](/assets/ios-corner-radius-settings.png)

And your output will be


![Output](/assets/ios-corner-radius-output.png)

Cheers ! 