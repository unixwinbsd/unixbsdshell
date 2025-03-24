---
title: Build Your Own Email Server on OpenBSD
date: "2025-02-06 09:17:10 +0300"
id: build-own-email-server-on-openbsd
lang: en
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "SysAdmin"
excerpt: Some public mail servers couldn’t send mail to some domains, e.g. protonmail.com due to country limitations. Hosting your own email server gives us the ability to manage such limitations
keywords: mail server, openbsd, freebsd, email, email server, private, public
---

If there is a need to create your own email server for your company or personal use for free (it’s actual for small businesses) so this information gives the point of interest. As a result, the mail can be sent over the world via your free mail server for the domain.  

Why could we make our own email server?<br><br/>
## 1.  Advantages of a private email server

### a. Our own mail domain
Sending a company’s mail from its domain is a good idea to be business attractive. E.g. sales@mycompany.com.

### b. Unlimited users
We can create as many users as we want without extra pay.

### c. Unlimited targeting
Unlimited target addresses while sending mail. In external paid services we have to pay per user.

### d. All the data is under our control
Nobody will be able to see our internal business conversation.

### e. Transmission control
Some public mail servers couldn’t send mail to some domains, e.g. protonmail.com due to country limitations. Hosting your own email server gives us the ability to manage such limitations.<br><br/>
##  2. Building a small, secure mail server with OpenBSD and OpenSMTPD
With the proliferation of fiber optic networks, many people are now trying to run their own email servers at home. Of course, if their ISP allows it - not all ISPs allow traffic to port 25 on home user connections. Given that many home fiber optic networks have become havens for spammers, this is understandable. However, many ISPs are willing to remove the filter on port 25 once the customer demonstrates that they are not running an open relay and generally know what they are doing.

So, the day comes when a true geek decides that he or she must run their own email server from home. Since I have built a few of these now, I thought it might help some people if I provided a blueprint on how to set one up for home use. <br><br/>
## 3. OpenSMTPD Installation guide for OpenBSD
This tutorial will guide you through the installation process of OpenSMTPD on OpenBSD. OpenSMTPD is an open source implementation of the Simple Mail Transfer Protocol. It is designed with security in mind and is easy to configure.

Note: This tutorial assumes that you are running OpenBSD.

To install OpenSMTPD on OpenBSD, follow the steps below:

**Update the package database:**

```
hostname1# pkg_add -u
```

After the update process is complete, you continue by installing OpenSMTPD.  
  
```
hostname1# pkg_add opensmtpd
```

Then you activate the OpenSMTPD service.  

```
hostname1# rcctl enable smtpd
hostname1# rcctl start smtpd
```

After that, you verify that the OpenSMTPD service is running.

```
hostname1# rcctl check smtpd
```
<br><br/>
## 4. Configuration OpenSMTPD
By default, OpenSMTPD is ready to use without any configuration. However, if you want to customize the configuration, you can modify the configuration file /etc/mail/smtpd.conf.

To edit the configuration file using the vi editor, run the following command.

```
hostname1# nano /etc/mail/smtpd.conf
```

After making changes to the configuration file, you will need to reload the OpenSMTPD service to apply the changes

```
hostname1# rcctl reload smtpd
```

This was just a quick example on how to setup your own simple email server. There are many more things that can be done, from adding redundancy with multiple mail servers, to doing DNSBL and bayesian analysis and filtering on incoming email to detect SPAM.
