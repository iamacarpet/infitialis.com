---
layout: post
title: Broken DKIM Signatures on Office365
date: 2017-12-14 15:40:00
categories:
  - office365
  - microsoft
---

It always came across funny before when&nbsp;[other people](http://www.schveiguy.com/blog/2017/05/how-to-report-a-bug-to-microsoft/)&nbsp;where having bad experiences with Microsoft support for Office365, leaving us thinking thank god that isn't us\!

Now we've found our own issue and wow, was that account accurate.

The support so far has been abysmal\!

I almost thought I was taking part in the new series of&nbsp;[Channel 4's FoneJacker](https://en.wikipedia.org/wiki/Fonejacker).

As a bit of background, we have split delivery with our MX pointing at Office365 and email for a select few users being sent to G Suite via a mail flow rule and an Outbound Gateway configured to point to Gmail.

We first noticed that delivery of messages with attachments from one of our 3rd party suppliers wasn't making it to the users on Gmail, but was making it to the users on Office365.

To make matters worse, the 3rd party supplier was getting a very misleading bounce message about SPF records, at first glance instructing them to add "[spf.protection.outlook.com](http://spf.protection.outlook.com/)" to their SPF.

//blockquote//

On further inspection, the actual error is hidden inside the email:

> *550 5.7.23 The message was rejected because of Sender Policy Framework violation -&gt; 550 5.7.1 Unauthenticated email from \*\*removed\*\*.co.uk is not accepted due to;domain's DMARC policy. Please contact the administrator of;\*\*removed\*\*.co.ukdomain if this was a legitimate mail. Please;visit;&nbsp;[https://support.google.com/mail/answer/2451690](https://support.google.com/mail/answer/2451690)&nbsp;to learn about the;DMARC initiative. d76si12884861pfk.321 - gsmtp*

After some help from Google's G Suite support (fabulous by the way, quick responses, helpful and extremely knowledgeable right from the off), we figured out that the issue was that the DKIM signatures are being broken as the messages transit Office365 and our 3rd party has a strict DMARC policy which requires all messages are validated by the receiving party for a valid signature.

&nbsp;

This only happens on multi-part messages with an attachment, as Microsoft are altering the message somehow and the problem with that, as pointed out by G Suite support, is that most of the body is part of the signature hash, so by altering it the signature hash no longer matches and the integrity of the message is compromised according to DKIM.

&nbsp;

While exploring the issue, we looked at the message that was successfully delievered to an Office365 user and the message body was also altered there too, but in a different way. Headers inside the multi-part sections had been added to do with spam handling, as well as given a Content-ID, etc.

&nbsp;

The SPAM headers look like this:

> X-Microsoft-Exchange-Diagnostics: \*\*REDACTED BASE64 DATA\*\*X-Microsoft-Antispam-Mailbox-Delivery: \*\*REDACTED STRING\*\*X-Microsoft-Antispam-Message-Info: \*\*REDACTED BASE64 DATA\*\*Content-ID: &lt;\*\*REDACTED\*\*@eurprd04.prod.outlook.com&gt;

On trying to re-create the issue, I sent an email from a gmail.com address to one of the accounts that is using split delivery, so I could see the issue from both sides. This identified something else, that Office365 is altering the multi-part boundry strings as it forwards on the message.

&nbsp;

To get to the bottom of the issue, I needed to play with the message until the signature did come back valid, to prove what the problem was, so I dropped the .eml files as .txt on a Linux box and got a copy of "[dkimpy-0.6.2](https://launchpad.net/dkimpy)/dkimverify.py" to validate the messages on the command line.

&nbsp;

It turns out, that hidden in the diff by the fact the multi-part boundary strings have changed, Office365 is adding a newline to the email between the boundaries, which is being included in the part of the body that is verified.

&nbsp;

The additional headers, the changes to the multi-part boundary strings and even the headers inside the multi-part sections themselves don't seem to affect the signature; it's just the newline.

&nbsp;

As you can see here, example diff from the email with attachment:

> 24,25c161,163&lt; --001a114733c035484b056025d07f--&lt; --001a114733c0354850056025d081---&gt; --001a114b306e43341b056025d01a--&gt;&gt; --001a114b306e43341e056025d01c

Compared to the email without the attachment:

> 26,27c88,89&lt; --001a1148e7ba4a6e8b05604cdd8c--&lt; --001a1148e7ba4a6e9105604cdd8e---&gt; --001a11468748570c3605604cdd36--&gt; --001a11468748570c3a05604cdd38

And a quick test showed that had fixed the validation:

> smelrose@vdev2:-$ python dkimpy-0.6.2/dkimverify.py &lt; broken\_email\_received.txtsignature verification failed
>
>
> smelrose@vdev2:-$ cp broken\_email\_received.txt br\_test.txt; vi br\_test.txt\[ edited the file and removed the newline \]
>
>
> smelrose@vdev2:-$ python dkimpy-0.6.2/dkimverify.py &lt; br\_test.txtsignature ok

So far, Microsoft support haven't understood the problem at all.

&nbsp;

After emailing them over the details of what I've been investigating, they insisted on calling me to discuss it over the phone, only to get confused and keep telling me over and over again to alter the SPF records.

&nbsp;

I reached out to secure@microsoft.com, who emailed back after a few hours to say if it wasn't a RCE or MITM, they weren't interested.

&nbsp;

My last attempt is to teach out to @Office365 on Twitter, who I will be sending a link to this article, so I'll update how that goes.