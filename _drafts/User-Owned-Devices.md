---
title: User Owned Devices - Towards Digital Sovereignty
categories:
  - Personal
tags:
  - TAG
published: true
header:
  image: mh/mh002.jpg
# comments:
#  host: social.wildeboer.net
#  username: jwildeboer
#  id: XXX 
---

This story starts with me thinking about getting a new gadget, just to play with it, take it apart, understand how it works, how it can fail, how it can be sabotaged. Nothing special. I like to do that kind of thing every now and then. My target this time: a fingerprint/RFID enabled doorlock.

## Finding the gadget

So I started looking at the various devices out there, after mentally noting some requirements:

- Not too expensive, as I will most likely break it in some way during the process anyway
- Doesn't have to be super secure and sophisticated, but also not irresponsibly bad (though that can be more fun :)
- Should be installable on my front door, so it needs a euro cylinder
- Ideally from a known brand (which typically means it fail on the first requirement of not being too expensive)
- Standalone operation is sufficient, no big networked/connected system needed as my apartment has just one front door.
- No loud motors, better to turn by hand. Not just because of noise but also because that tactile approach seems a bit more secure and less error-prone.

I saw a few names popping up all the time. The [Welock](https://www.welock.com) brand, [Tapkey](https://shop.tapkey.com/collections/digital-locks) and [Bold](https://boldsmartlock.com/products/). So I started comparing them and also looked at some lesser known brands like Elinksmart and Avatto.

## A pattern emerges

While comparing features and stuff, I noticed something odd. Almost all "smart" door locks need a smartphone, an app and an account on some proprietary service. And quite a lot of them use the same backend system, regardless of the brand name - [Tuya](https://www.tuya.com).

![Looking for "door lock tuya" on Ali Express](/images/2023/12/locks01.jpg)
**Looking for "door lock tuya" on Ali Express**

Only one of the locks I looked at can work completely offline and standalone. Yes, you can also use an app and yes, that also uses Tuya, but you can work with the lock without the app. It's the [Welock SECBN51](https://www.welock.com/collections/smart-lock-eu/products/welock-fingerprint-electronic-smart-door-lock-cylinder-secbn51)

![Welock SECBN51](/images/2023/12/welock01.jpg)
**The Welock SECBN51**

So I ordered one. Should arrive tomorrow. Will be fun!

## BUT

But all of this research led to me to a bunch of thoughts on the state of these devices and the ecosystms behind it. Tuya is a centralised service behind a lot of different brands. This creates an uneasy SPoF (Single point of Failure) feeling, especcially when you look at their latest [investor report from 2023-11-29](https://s27.q4cdn.com/751054641/files/doc_presentations/2023/Nov/29/tuya-23q3-presentation_vff.pdf) where they say they are now getting close to break-even (meaning: they didn't make money so far) by reducing their workforce significantly (less people on the ground while they add more and more brands and devices).

![Tuya investor report 2023-11-29](/images/2023/12/Tuya01.jpg)
**Tuya investor report 2023-11-29, slide 19**

These rather unkown giants in the background always make me nervous and a question keeps nagging in me, getting louder every day. Is there a better, more decentralised way?

