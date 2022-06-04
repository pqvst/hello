---
layout: post
title: Fastmail Spam Learning is Weird
tags: [Email, Fastmail, Spam]
---

I recently moved my email from Google Workspace to Fastmail. For the most part it's been going great. However, I have definitely seen an increase in spam hitting my inbox. With Google Workspace I pretty much *never* received any Spam in my inbox. With Fastmail I receive a couple of messages every week.

Here are some examples (I'm glad to see that at least the Spam score reported by Fastmail is improving since I first received these messages):

![](/assets/img/fastmail-spam/spam1.png)

![](/assets/img/fastmail-spam/spam2.png)

![](/assets/img/fastmail-spam/spam4.png)

![](/assets/img/fastmail-spam/spam5.png)

Since I use Spark as my email client on both macOS and Android, I would simply hit "Mark as Spam" and hoped that would eventually make things better. Except, I started to notice the same Spam messages hitting my inbox over and over again. Weird...

![](/assets/img/fastmail-spam/repeat-spam.png)

After looking into how Fastmail handles Spam, and talking with their customer support (which was great) I eventually learned that **simply moving an email to the Spam folder with a third party email client <u>does not</u> actually report the message as spam**. It *just* moves the email to the spam folder. 

[Fastmail: Report spam/non-spam on email clients](https://www.fastmail.help/hc/en-us/articles/1500000278142-Improving-spam-protection#spamfolders)

Instead, the recommended approach is to setup your own spam folder, on which you can then enable "Spam learning". Note that Fastmail does not recommend enabling spam learning on your default Spam folder since that can cause a "false positive feedback loop".

![](/assets/img/fastmail-spam/spamx-folder.png)

To work around this, I created a new folder called SpamX (for the lack of a better name). Under "Show advanced preferences", make sure you enable the option called "Scan this folder daily and learn any new messages" and set it to "as spam".

![](/assets/img/fastmail-spam/learn-spam-setting.png)

Then, in your email client (in my case Spark), change your spam folder to this new folder. Now, in theory, any time I receive any Spam in my inbox and mark it as spam, Fastmail will eventually learn about these.

![](/assets/img/fastmail-spam/spark-spam-folder.png)

---

I'm really surprised Fastmail doesn't solve this in a better way. It would be great if Fastmail would just automatically learn Spam on any message that is manually moved to the default Spam folder. That really doesn't seem like such a difficult thing for them to do.