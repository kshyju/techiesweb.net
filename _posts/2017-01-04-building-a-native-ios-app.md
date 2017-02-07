---
layout: post
title: Building a native IOS app
date: 2017-01-04 00:11:29.000000000 -05:00 
tags: [IOS, Mobile]
---

Last year, I built a native IOS app from scratch with no prior experience with native app development.

This was a side project with a friend to build a social diet sharing app. We started with the android app initially, but later decided to build an IOS version of it. 

Since I built the initial versions of the android app, I had a very good notion of the UI i was going to build (consistency is important. I spent a good amount of time for UI layouts for  android app). The next question was what platform we want to build it ? We originally had hybrid vs native discussion and  I convinced him to go for the native way. Coming from the C#/Typescript background, the choice between learning Objective C vs Swift was obvious.


Learning  a new language
====

Swift 2 looked a lot like Typescript, with which I fell in love around that time. So picking up the language syntax was easy. With internet, learning a technology is not hard, it is the patience and persistence until you get what you want, takes you forward. With the help of internet i was able to build the initial version with the following basic features

1. Login / Register
2. Feed
3. User profile
4. Search 

App store wait time
====
After submitting the first version of the IOS app to the store, I kept refreshing my email every other hour checking for a response! When you are so eager(I was so eager to tell people our IOS app is out), thing happens very slowly! It took 7 days for me to get a response and it was not the one i was waiting for :( The appstore team rejected the app due to many reasons.

1. There was no "flag post" feature
2. Lack of "Block user" feature
3. "Terms & Conditions" and "Privacy Policy" was missing.


I fixed the issues quickly and resubmitted. After another 6 days, i finaly got what i was waiting for.

![Gravatar tag helper][1]

[1]: /assets/Caloric-approved.png

I shipped multiple updates to the app over the next few months including features like

1. Health Rating
2. Taste Rating
3. Public Feed
4. Badges
5. Push notifications
6. Like & Comments on posts
7. Locations ( Using 4Square API)
8. Post Category
9. Ingredients & Calorie information on post
10. Facebook & Twitter Integration


Where is the app now ? 
====

We decided to not continue maintaining the IOS app in the store anymore. So it is not available to download. I created a video of the app before deleting it from my phone. You can see it here. (It is around 10 minutes. Sorry! My video production skills are so poor :))

<iframe width="700" height="495" src="http://www.youtube.com/embed/duDTEDQrYaU" frameborder="0" allowfullscreen></iframe>

I really enjoyed building the native app from scratch. I plan on sharing the useful code snippets & tricks on swift / IOS development.

Cheers



