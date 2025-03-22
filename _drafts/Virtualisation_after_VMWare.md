---
title: Virtualisation After VMWare
categories:
  - Personal
tags:
  - Linux
  - KVM
  - OpenShift
published: true
header:
  image: mh/mh002.jpg
comments:
  host: social.wildeboer.net
  username: jwildeboer
  id: 114166126991847944
---

On modern [#Linux](https://social.wildeboer.net/tags/Linux), a virtual machine using [#KVM](https://social.wildeboer.net/tags/KVM) is more or less just another running process. Which allows you to treat a VM more or less as a container. Which is why we at Red Hat offer Openshift Virtualisation, that puts all of that together. This was kinda mind-blowing for some at the event where I presented yesterday. Seeing VMs as "fat" containers that you can now start planning to convert to real containers :)

But this is the way we look at things. What abstraction layers make sense? And how can we implement them gradually to get to better solutions? With the goal of simplification instead of adding complexity. This is one of the advantages of having an open source development model, IMHO.

This is how I look at the "Virtualisation after VMWare" issue. It's not about staying in the past by creating a drop-in alternative solution. It's about thinking hard on where virtualisation fits into the future. And this hard work is cooperative. Our customers are part of this thought process. They also need to invest in rethinking their approach. And when we do this together, magic happens. That's the Open Source Way, IMHO.
  
Disclaimer for those that don't know: I work at Red Hat. My job title is EMEA Evangelist since 17 years. I have the luxury to think outside of quarterly revenue targets and muse about big pictures. But what I say here is based on 30 years of experience in Open Source and Free Software, 20 years of that at Red Hat, so I also know the business side. I hope my musings are helpful. Yesterday, after my presentation, the feedback definitely indicated that it was for quite some in the audience :)

**This post was originally a [thread](https://social.wildeboer.net/@jwildeboer/114166126991847944) on Mastodon**