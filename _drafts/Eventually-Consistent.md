---
# title: Eventually Consistent, the forgotten third option of the CAP theorem
categories:
  - Personal
tags:
  - TAG
published: false
header:
  image: mh/mh002.jpg
# comments:
#  host: social.wildeboer.net
#  username: jwildeboer
#  id: XXX 
---

"For a moment there, Lotus Notes appeared to do everything a company needed" [https://www.theregister.com/2024/01/19/remembering\_lotus\_notes/](https://www.theregister.com/2024/01/19/remembering_lotus_notes/)

Yes, it did. Around V4.x. And it was a wonderful time. (I was a certified Lotus Notes Admin and Dev at the time) All patents on their sync approach should have expired by now, maybe it's time to bring it back as Open Source?

With the knowledge of today, Lotus Notes starts to look like a cool git setup, with everyone having their local repos that they sync when possible, but that always work regardless of the last sync and a modern UI on top that hides all the details and gives you a low/zero-code approach to app development.

It was far, far ahead of its time, IMHO. But we can learn a lot, a lot lot, from how it worked and how we can use modern, open source tools to get back to where we were in those golden days ...

It might even use [#ActivityPub](https://social.wildeboer.net/tags/ActivityPub) under the hood ;)

Think distributed, peer to peer git commits/rebases exchanging ActivityPub messages.

Or the other way round. I hope some of you get the idea. Too many people still think that git remotes are a n:1 relation (meaning many local repos syncing with one single authoritative remote), where in reality they can be m:n by design.

Lotus Notes was built on this model of "eventually consistent", that tolerated and was able to manage a lot of inconsistencies under the hood, with users never noticing.

Compare that to conflict handling that is typically blocking in many solutions and needs some kind of resolving with potential data loss.

Too many people have interpreted the CAP theorem [1] as being binary. Either strongly consistent or highly available.

Decentralised thinking however offers a third point of view. Lazy, relaxed or eventual consistency that accepts network partitioning not as an exception but as a rule [2].

Lotus Notes took that mostly unexplored road a long time ago. And I think it is worth discussing again in times of [#ActivityPub](https://social.wildeboer.net/tags/ActivityPub) and state censorship.

You have to remember that Lotus Notes was architected at a time of "networks of networks" where the fight on who would win the WAN war (Wide Area Networks) wasn't decided. We were using modems (later ISDN cards) to connect directly to our servers. It later eventually became TCP/IP and the internet that won.

And now, with deep packet inspection, centralised services built on TCP/IP, we do see a glimmer of a renaissance of "networks of networks" and the concept of eventually consistent becomes relevant again, IMHO. 

But maybe I am just thinking too much :) 

[1] [https://en.wikipedia.org/wiki/CAP\_theorem](https://en.wikipedia.org/wiki/CAP_theorem)
[2] [https://core.ac.uk/download/pdf/237193592.pdf](https://core.ac.uk/download/pdf/237193592.pdf)