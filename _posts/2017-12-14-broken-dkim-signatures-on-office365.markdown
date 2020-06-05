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

<blockquote style="padding: 10px;">
<br>
<div style="-webkit-text-stroke-width: 0px; background-color: white; color: #222222; font-family: arial, sans-serif; font-size: 12.8px; font-style: normal; font-variant-caps: normal; font-variant-ligatures: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-decoration-color: initial; text-decoration-style: initial; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px;">
<table cellpadding="0" cellspacing="0" style="border-collapse: collapse; color: black; max-width: 548px; padding-bottom: 0px; padding-top: 0px; width: 548px;"><tbody>
<tr><td style="font-family: arial, sans-serif; margin: 0px; padding-bottom: 20px;"><img class="m_-6114787014011996939gmail-CToWUd CToWUd" height="28" src="https://ci3.googleusercontent.com/proxy/LizBT1Ous4Eielab03jQSa54qs_Q-wq1aPwBuJF0nwKczAJPeaYRL0byn1Is8um5qGIMPxbJOw8jkpkAaNO5o4E8fjhDMIRownYe-gha2mCmFTNOO7YKHk1N4KcX1cv8yYcTCrrIMleiK5OlkS15Js3aX4VYwpJPprc8y73akcUCYuaV8do=s0-d-e1-ft#http://products.office.com/en-us/CMSImages/Office365Logo_Orange.png?version=b8d100a9-0a8b-8e6a-88e1-ef488fee0470" style="max-width: 100%;" width="126"></td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 16px; margin: 0px; padding-bottom: 10px;">Your message to&nbsp;<span style="color: #0072c6;"><a href="mailto:sam.m----@a1comms.com" style="color: #1155cc;" target="_blank">sam.m****@a1comms.com</a></span>&nbsp;<wbr>couldn't be delivered.</td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 24px; margin: 0px; padding-bottom: 20px; padding-top: 0px; text-align: center;"><span style="color: #0072c6;"><a data-saferedirecturl="https://www.google.com/url?hl=en&amp;q=http://a1comms.com/&amp;source=gmail&amp;ust=1513337851857000&amp;usg=AFQjCNFAhHoQLVjbARdbvEJQMnKbW67Jkg" href="http://a1comms.com/" style="color: #1155cc;" target="_blank">a1comms.com</a></span>&nbsp;couldn't confirm that your message was sent from a trusted location.</td></tr>
<tr><td style="font-family: arial, sans-serif; margin: 0px; padding-bottom: 15px; padding-left: 0px; padding-right: 0px;"><table style="border-collapse: collapse; font-weight: 600; max-width: 548px; padding-bottom: 0px; padding-top: 0px;"><tbody>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 15px; margin: 0px; vertical-align: bottom; width: 181px;"><span style="color: white;"><span style="color: black;">****.******</span></span></td><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 15px; margin: 0px; text-align: center; vertical-align: bottom; width: 186px;"><span style="color: white;"><span style="color: black;">Office 365</span></span></td><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 15px; margin: 0px; text-align: right; vertical-align: bottom; width: 181px;"><span style="color: white;"><span style="color: black;">sam.m****</span></span></td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 14px; font-weight: 400; margin: 0px; padding-bottom: 0px; padding-top: 0px; vertical-align: middle; width: 181px;"><span style="color: white;"><span style="color: #c00000;"><b>Action Required</b></span></span></td><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 14px; font-weight: 400; margin: 0px; padding-bottom: 0px; padding-top: 0px; text-align: center; vertical-align: middle; width: 186px;"><br></td><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 14px; font-weight: 400; margin: 0px; padding-bottom: 0px; padding-top: 0px; text-align: right; vertical-align: middle; width: 181px;"><span style="color: white;"><span style="color: black;">Recipient</span></span></td></tr>
<tr><td colspan="3" style="font-family: arial, sans-serif; margin: 0px; padding: 0px;"><table cellpadding="0" cellspacing="0" style="border-collapse: collapse; padding: 0px;"><tbody>
<tr height="10"><td bgcolor="#c00000" height="10" style="font-family: arial, sans-serif; font-size: 6px; height: 10px; line-height: 10px; margin: 0px; padding: 0px; width: 180px;" width="180"><br></td><td bgcolor="#ffffff" height="10" style="font-family: arial, sans-serif; font-size: 6px; height: 10px; line-height: 10px; margin: 0px; padding: 0px; width: 4px;" width="4"><br></td><td bgcolor="#cccccc" height="10" style="font-family: arial, sans-serif; font-size: 6px; height: 10px; line-height: 10px; margin: 0px; padding: 0px; width: 180px;" width="180"><br></td><td bgcolor="#ffffff" height="10" style="font-family: arial, sans-serif; font-size: 6px; height: 10px; line-height: 10px; margin: 0px; padding: 0px; width: 4px;" width="4"><br></td><td bgcolor="#cccccc" height="10" style="font-family: arial, sans-serif; font-size: 6px; height: 10px; line-height: 10px; margin: 0px; padding: 0px; width: 180px;" width="180"><br></td></tr>
</tbody></table>
</td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 14px; font-weight: 400; line-height: 20px; margin: 0px; padding: 0px; width: 181px;"><span style="color: white;"><span style="color: #c00000;">SPF validation error</span></span></td><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 14px; font-weight: 400; line-height: 20px; margin: 0px; padding: 0px; text-align: center; width: 186px;"><br></td><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 14px; font-weight: 400; line-height: 20px; margin: 0px; padding: 0px; text-align: right; width: 181px;"><br></td></tr>
</tbody></table>
</td></tr>
<tr><td style="font-family: arial, sans-serif; margin: 0px; padding-left: 10px; padding-right: 10px; padding-top: 0px; width: 528px;"><br>
<table style="background-color: #f2f5fa; margin-left: 0px; padding: 0px; width: 528.8px;"><tbody>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 21px; margin: 0px; padding: 0px 10px;">How to Fix It</td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 16px; margin: 0px; padding: 0px 10px 6px;">Your organization's email admin will have to diagnose and fix your domain's email settings. Please forward this message to your email admin.</td></tr>
</tbody></table>
</td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 14px; margin: 0px; padding-bottom: 4px; padding-top: 10px;"><br>
<i></i></td></tr>
<tr><td style="font-family: arial, sans-serif; font-size: 0px; line-height: 0px; margin: 0px; padding-bottom: 0px; padding-top: 0px;"><hr>
</td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 21px; margin: 0px;"><br>
More Info for Email Admins</td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 14px; margin: 0px;"><i>Status code: 550 5.7.23</i><br>
<br>
This error occurs when Sender Policy Framework (SPF) validation for the sender's domain fails. If you're the sender's email admin, make sure the SPF records for your domain at your domain registrar are set up correctly. Office 365 supports only one SPF record (a TXT record that defines SPF) for your domain. Include the following domain name:&nbsp;<b><a data-saferedirecturl="https://www.google.com/url?hl=en&amp;q=http://spf.protection.outlook.com/&amp;source=gmail&amp;ust=1513337851857000&amp;usg=AFQjCNET638unV90GMuTHYLy5hog0_tuxg" href="http://spf.protection.outlook.com/" style="color: #1155cc;" target="_blank">spf.protection.outlook.<wbr>com</a></b>. If you have a hybrid configuration (some mailboxes in the cloud, and some mailboxes on premises) or if you're an Exchange Online Protection standalone customer, add the outbound IP address of your on-premises servers to the TXT record.<br>
<br>
For more information and instructions about configuring SPF records see&nbsp;<a data-saferedirecturl="https://www.google.com/url?hl=en&amp;q=https://technet.microsoft.com/library/dn789058(v%3Dexchg.150).aspx&amp;source=gmail&amp;ust=1513337851858000&amp;usg=AFQjCNEqTcKx3fK-U9jpKfqVNpHBpo8fxg" href="https://technet.microsoft.com/library/dn789058(v=exchg.150).aspx" style="color: #1155cc;" target="_blank">Customize an SPF record to validate outbound mail sent from your domain</a>&nbsp;and also&nbsp;<a data-saferedirecturl="https://www.google.com/url?hl=en&amp;q=https://support.office.com/article/External-Domain-Name-System-records-for-Office-365-c0531a6f-9e25-4f2d-ad0e-a70bfef09ac0%23BKMK_SPFrecords&amp;source=gmail&amp;ust=1513337851858000&amp;usg=AFQjCNG7QDmiW89lDlZFSlzcKsxSd1jwpA" href="https://support.office.com/article/External-Domain-Name-System-records-for-Office-365-c0531a6f-9e25-4f2d-ad0e-a70bfef09ac0#BKMK_SPFrecords" style="color: #1155cc;" target="_blank">External Domain Name System records for Office 365</a>.</td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 17px; margin: 0px;">Original Message Details</td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 14px; line-height: 20px; margin: 0px;"><table style="border-collapse: collapse; margin-left: 10px; width: 548px;"><tbody>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; margin: 0px; white-space: nowrap; width: 140px;" valign="top">Created Date:</td><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; margin: 0px;">12/12/2017 2:10:29 PM</td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; margin: 0px; white-space: nowrap; width: 140px;" valign="top">Sender Address:</td><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; margin: 0px;"><a href="mailto:----.----@cloud----.co.uk" style="color: #1155cc;" target="_blank">****.****@cloud****.co<wbr>.uk</a></td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; margin: 0px; white-space: nowrap; width: 140px;">Recipient Address:</td><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; margin: 0px;"><a href="mailto:sam.m----@a1comms.com" style="color: #1155cc;" target="_blank">sam.m****@a1comms.com</a></td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; margin: 0px; white-space: nowrap; width: 140px;">Subject:</td><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; margin: 0px;">Re: FW: **REDACTED**</td></tr>
</tbody></table>
</td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 17px; margin: 0px;"><br>
Error Details</td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 14px; line-height: 20px; margin: 0px;"><table style="border-collapse: collapse; margin-left: 10px; width: 548px;"><tbody>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; margin: 0px; white-space: nowrap; width: 140px;" valign="top">Reported error:</td><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; margin: 0px;"><i>550 5.7.23 The message was rejected because of Sender Policy Framework violation -&gt; 550 5.7.1 Unauthenticated email from **REDACTED**.co.uk&nbsp;is not accepted due to;domain's DMARC policy. Please contact the administrator of;**REDACTED**.co.ukdomain if this was a legitimate mail. Please;visit;&nbsp;<a data-saferedirecturl="https://www.google.com/url?hl=en&amp;q=https://support.google.com/mail/answer/2451690&amp;source=gmail&amp;ust=1513337851858000&amp;usg=AFQjCNEzS3mGvu8px9FVNYyr4UVsEKBGPg" href="https://support.google.com/mail/answer/2451690" style="color: #1155cc;" target="_blank">https://support.<wbr>google.com/mail/answer/2451690</a><wbr>&nbsp;to learn about the;DMARC initiative. d76si12884861pfk.321 - gsmtp</i></td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; margin: 0px; white-space: nowrap; width: 140px;">DSN generated by:</td><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; margin: 0px;"><a data-saferedirecturl="https://www.google.com/url?hl=en&amp;q=http://vi1pr0401mb2302.eurprd04.prod.outlook.com/&amp;source=gmail&amp;ust=1513337851858000&amp;usg=AFQjCNH2GwX8FXkqs19iFSdcNKxf-Ra9Ng" href="http://vi1pr0401mb2302.eurprd04.prod.outlook.com/" style="color: #1155cc;" target="_blank">VI1PR0401MB2302.eurprd04.prod.<wbr>outlook.com</a></td></tr>
<tr><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; margin: 0px; white-space: nowrap; width: 140px;">Remote server:</td><td style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; margin: 0px;"><a data-saferedirecturl="https://www.google.com/url?hl=en&amp;q=http://mx.google.com/&amp;source=gmail&amp;ust=1513337851858000&amp;usg=AFQjCNEPbZlq9_VPg1zZghhWoFHuPfXnmA" href="http://mx.google.com/" style="color: #1155cc;" target="_blank">mx.google.com</a></td></tr>
</tbody></table>
</td></tr>
</tbody></table>
<br>
<table cellspacing="0" style="width: 880px;"><tbody>
<tr><td colspan="6" style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 17px; line-height: 20.4px; margin: 0px; padding-bottom: 4px; padding-top: 4px;">Message Hops</td></tr>
<tr><td style="background-color: #f2f5fa; border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px; white-space: nowrap;">HOP</td><td style="background-color: #f2f5fa; border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px; white-space: nowrap; width: 80px;">TIME (UTC)</td><td style="background-color: #f2f5fa; border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px; white-space: nowrap;">FROM</td><td style="background-color: #f2f5fa; border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px; white-space: nowrap;">TO</td><td style="background-color: #f2f5fa; border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px; white-space: nowrap;">WITH</td><td style="background-color: #f2f5fa; border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px; white-space: nowrap;">RELAY TIME</td></tr>
<tr><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px; text-align: center;">1</td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px; width: 80px;">12/12/2017<br>
2:10:29 PM</td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;"><br></td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;">10.200.17.150</td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;">HTTP</td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;">*</td></tr>
<tr><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px; text-align: center;">2</td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px; width: 80px;">12/12/2017<br>
2:10:48 PM</td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;"><br></td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;"><a data-saferedirecturl="https://www.google.com/url?hl=en&amp;q=http://mail-qt0-f169.google.com/&amp;source=gmail&amp;ust=1513337851858000&amp;usg=AFQjCNHH_R4wYCHrE1512Lp8B1HnegR30g" href="http://mail-qt0-f169.google.com/" style="color: #1155cc;" target="_blank">mail-qt0-f169.google.com</a></td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;">SMTP</td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;">19&nbsp;sec</td></tr>
<tr><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px; text-align: center;">3</td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px; width: 80px;">12/12/2017<br>
2:10:48 PM</td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;"><a data-saferedirecturl="https://www.google.com/url?hl=en&amp;q=http://mail-qt0-f169.google.com/&amp;source=gmail&amp;ust=1513337851858000&amp;usg=AFQjCNHH_R4wYCHrE1512Lp8B1HnegR30g" href="http://mail-qt0-f169.google.com/" style="color: #1155cc;" target="_blank">mail-qt0-f169.google.com</a></td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;"><a data-saferedirectreason="2" data-saferedirecturl="https://www.google.com/url?hl=en&amp;q=http://he1eur01ft024.mail.protection.outlook.com/&amp;source=gmail&amp;ust=1513337851858000&amp;usg=AFQjCNHeKjvQXHWHL_hjoZXd_YhipNb2lQ" href="http://he1eur01ft024.mail.protection.outlook.com/" style="color: #1155cc;" target="_blank">HE1EUR01FT024.mail.protection.<wbr>outlook.com</a></td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;">Microsoft SMTP Server (version=TLS1_2, cipher=TLS_ECDHE_RSA_WITH_AES_<wbr>256_CBC_SHA_P384)</td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;">*</td></tr>
<tr><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px; text-align: center;">4</td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px; width: 80px;">12/12/2017<br>
2:10:48 PM</td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;"><a data-saferedirectreason="2" data-saferedirecturl="https://www.google.com/url?hl=en&amp;q=http://he1eur01ft024.eop-eur01.prod.protection.outlook.com/&amp;source=gmail&amp;ust=1513337851858000&amp;usg=AFQjCNEUAaW-e5iqrDRXSFAbCP5IM8tH5A" href="http://he1eur01ft024.eop-eur01.prod.protection.outlook.com/" style="color: #1155cc;" target="_blank">HE1EUR01FT024.eop-EUR01.prod.p<wbr>rotection.outlook.com</a></td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;"><a data-saferedirecturl="https://www.google.com/url?hl=en&amp;q=http://vi1pr04ca0068.outlook.office365.com/&amp;source=gmail&amp;ust=1513337851858000&amp;usg=AFQjCNGbPg9MSqgvMRKcKXg-MeswZNNvLA" href="http://vi1pr04ca0068.outlook.office365.com/" style="color: #1155cc;" target="_blank">VI1PR04CA0068.outlook.office36<wbr>5.com</a></td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;">Microsoft SMTP Server (version=TLS1_2, cipher=TLS_ECDHE_RSA_WITH_AES_<wbr>256_CBC_SHA384)</td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;">*</td></tr>
<tr><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px; text-align: center;">5</td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px; width: 80px;">12/12/2017<br>
2:10:48 PM</td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;"><a data-saferedirecturl="https://www.google.com/url?hl=en&amp;q=http://vi1pr04ca0068.eurprd04.prod.outlook.com/&amp;source=gmail&amp;ust=1513337851858000&amp;usg=AFQjCNGwVPiC0z4xehN9z_Zsc7jjAxgPGQ" href="http://vi1pr04ca0068.eurprd04.prod.outlook.com/" style="color: #1155cc;" target="_blank">VI1PR04CA0068.eurprd04.prod.ou<wbr>tlook.com</a></td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;"><a data-saferedirecturl="https://www.google.com/url?hl=en&amp;q=http://vi1pr0401mb2302.eurprd04.prod.outlook.com/&amp;source=gmail&amp;ust=1513337851858000&amp;usg=AFQjCNH2GwX8FXkqs19iFSdcNKxf-Ra9Ng" href="http://vi1pr0401mb2302.eurprd04.prod.outlook.com/" style="color: #1155cc;" target="_blank">VI1PR0401MB2302.eurprd04.prod.<wbr>outlook.com</a></td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;">Microsoft SMTP Server (version=TLS1_2, cipher=TLS_ECDHE_RSA_WITH_AES_<wbr>256_CBC_SHA384_P256)</td><td style="border-bottom: 1px solid rgb(153, 153, 153); font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 12px; margin: 0px; padding: 8px;">*</td></tr>
</tbody></table>
<div style="font-family: &quot;Segoe UI&quot;, Frutiger, Arial, sans-serif; font-size: 17px; margin-bottom: 5px; margin-top: 19px; padding-bottom: 0px; padding-top: 4px;">
Original Message Headers</div>
** REDACTED MESSAGE HEADERS **
<br>
<br>
Final-Recipient: rfc822;sam.m****@a1comms.com<br>
Action: failed<br>
Status: 5.7.23<br>
Diagnostic-Code: smtp;550 5.7.23 The message was rejected because of Sender Policy Framework violation -&gt; 550 5.7.1 Unauthenticated email from **REDACTED**.co.uk is not accepted due to;domain's DMARC policy. Please contact the administrator of;**REDACTED**.co.uk domain if this was a legitimate mail. Please;visit; <a href="https://support.google.com/mail/answer/2451690">https://support.google.com/mail/answer/2451690</a> to learn about the;DMARC initiative. d76si12884861pfk.321 - gsmtp<br>
Remote-MTA: dns;<a href="http://mx.google.com/">mx.google.com</a><br>
X-Display-Name: Samuel Melrose</div>
</blockquote>

On further inspection, the actual error is hidden inside the email:

> *550 5.7.23 The message was rejected because of Sender Policy Framework violation -&gt; 550 5.7.1 Unauthenticated email from \*\*removed\*\*.co.uk is not accepted due to;domain's DMARC policy. Please contact the administrator of;\*\*removed\*\*.co.ukdomain if this was a legitimate mail. Please;visit;&nbsp;[https://support.google.com/mail/answer/2451690](https://support.google.com/mail/answer/2451690)&nbsp;to learn about the;DMARC initiative. d76si12884861pfk.321 - gsmtp*

After some help from Google's G Suite support (fabulous by the way, quick responses, helpful and extremely knowledgeable right from the off), we figured out that the issue was that the DKIM signatures are being broken as the messages transit Office365 and our 3rd party has a strict DMARC policy which requires all messages are validated by the receiving party for a valid signature.

This only happens on multi-part messages with an attachment, as Microsoft are altering the message somehow and the problem with that, as pointed out by G Suite support, is that most of the body is part of the signature hash, so by altering it the signature hash no longer matches and the integrity of the message is compromised according to DKIM.

While exploring the issue, we looked at the message that was successfully delievered to an Office365 user and the message body was also altered there too, but in a different way. Headers inside the multi-part sections had been added to do with spam handling, as well as given a Content-ID, etc.


The SPAM headers look like this:

> X-Microsoft-Exchange-Diagnostics: \*\*REDACTED BASE64 DATA\*\*X-Microsoft-Antispam-Mailbox-Delivery: \*\*REDACTED STRING\*\*X-Microsoft-Antispam-Message-Info: \*\*REDACTED BASE64 DATA\*\*Content-ID: &lt;\*\*REDACTED\*\*@eurprd04.prod.outlook.com&gt;

On trying to re-create the issue, I sent an email from a gmail.com address to one of the accounts that is using split delivery, so I could see the issue from both sides. This identified something else, that Office365 is altering the multi-part boundry strings as it forwards on the message.


To get to the bottom of the issue, I needed to play with the message until the signature did come back valid, to prove what the problem was, so I dropped the .eml files as .txt on a Linux box and got a copy of "[dkimpy-0.6.2](https://launchpad.net/dkimpy)/dkimverify.py" to validate the messages on the command line.

It turns out, that hidden in the diff by the fact the multi-part boundary strings have changed, Office365 is adding a newline to the email between the boundaries, which is being included in the part of the body that is verified.

The additional headers, the changes to the multi-part boundary strings and even the headers inside the multi-part sections themselves don't seem to affect the signature; it's just the newline.


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