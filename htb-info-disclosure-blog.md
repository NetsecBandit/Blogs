# I Found an Information Disclosure Bug in Hack The Box — Here's What Happened

*By Saim Raza (NetsecBandit)*

---

![COAE Badge](your-badge-image-here.png)

---

I was not actively hunting for bugs on HTB. I was just using the platform — doing what I normally do — when something felt off. A response came back with data it probably shouldn't have. That's usually how these things go. Not a planned recon session. Just noticing something that doesn't look right.

## What I Found

The vulnerability was an information disclosure issue in the Hack The Box platform. Information disclosure bugs don't always get the attention that RCE or SQLi do, but they matter. Exposed data can feed into bigger attacks — user enumeration, targeted phishing, internal structure mapping. The severity depends on what's leaking and to whom.

I'm keeping the technical specifics vague here out of respect for HTB's responsible disclosure process. The bug has since been reported and addressed.

## Reporting It

I reported it through HTB's official responsible disclosure channel. Wrote up a clear proof of concept, described the impact, and waited. I've reported bugs before where you send a detailed writeup into the void and hear nothing. HTB wasn't like that.

They responded, confirmed the issue, and rewarded me with a **COAE (Certified Offshore Assault Expert) voucher**. I wasn't expecting that. It's a solid cert, and getting it as a reward for a bug report felt good in a way that a generic "thanks" email doesn't.

## Why Bother Reporting It?

Honestly, because I use the platform. HTB is where I practice, where I learn, where I spent a lot of hours. Finding a bug and just keeping quiet about it — or worse, exploiting it — didn't feel right.

There's also a practical side: responsible disclosure builds trust. If you find something, report it cleanly, and get recognized for it, that's a better outcome for everyone involved. The cert is nice, but not the main point.

## Takeaway

Bug hunting doesn't always happen when you're hunting for bugs. Sometimes it happens because you've spent enough time with systems that something slightly wrong catches your eye before you consciously register why.

If you're doing CTFs, pentesting labs, or just spending time on platforms like HTB — pay attention to the responses you're getting. Not just whether the challenge works, but whether the application is behaving the way you'd expect it to.

That's usually where the real stuff is.

---

*Feel free to reach out on LinkedIn or HTB if you want to talk about bug bounty or responsible disclosure.*
