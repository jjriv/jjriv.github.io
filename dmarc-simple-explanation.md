Title: A Simplified Explanation of DMARC
Slug: dmarc-simplified
Date: 2024-05-28
Category: articles
Tags: DMARC
Summary: To help understand how DMARC compliance works, and why SPF and/or DKIM alignment is necessary, I find that a postal service analogy works best.

##What is DMARC?

DMARC, which stands for [Domain-based Message Authentication, Reporting & Conformance](https://dmarc.org/), is an email authentication protocol designed to give email senders the ability to prove ownership of their domain. 

To be DMARC compliant, an email sender must correctly implement at least one of two protocols — SPF ([Sender Policy Framework](https://postmarkapp.com/guides/spf)) and DKIM ([DomainKeys Identified Mail](https://postmarkapp.com/guides/dkim)). Now that we've gotten all the acronyms out of the way, let's move on to why DMARC is important. 

The simple truth is that email senders who are not DMARC compliant will find their emails [blocked by Gmail and Yahoo](https://meetmarigold.com/blogs/2024-guide-to-google-yahoos-new-privacy-protections/), and junked by many other mail providers. 

SPF and DKIM are two different methodologies, however, they have the same outcome. When implemented correctly they provide proof that your emails were sent by you, not an impostor. 

To help understand how they work, and why they are important, I find that a postal service analogy works best. Let's assume you want to mail a letter to one of your customers the old fashioned way, with an envelope and a stamp. Here's how SPF and DKIM would help ensure it gets delivered.

##SPF

SPF provides a way for you to broadcast the address of your local post office to your customer, designating them as a safe deliverer. When you drop off the letter for delivery, the postal clerk stamps the envelope with their address. This allows your customer's mail person to verify your letter before putting it in their mailbox by comparing the stamped address on the envelope with the address you've made public. 

However, there is a loophole that a bad actor could easily exploit. Because you've broadcasted the address of your local post office, anyone can send a letter on your behalf simply by dropping it off at the same location. To prevent this from happening, your letter must also be authenticated by the post office using a method called SPF alignment.

To authenticate your letter, the postal clerk first checks your identification. Once they've verified you as the rightful sender, they use a different stamp — one that still includes their address, but is unique to you. Before your customer's mail person drops off your letter they will now perform two checks. In addition to verifying that the stamped address matches the address you broadcasted, they will also make sure the stamp is the same unique one used by your local post office. 

With SPF alignment implemented correctly, the letter can't be sent by anyone other than you. 

##DKIM

DKIM is similar to SPF. But, instead of broadcasting the post office address, you broadcast a sample of your handwriting. When you drop off the letter for delivery, you sign the envelope with your signature. Your customer's mail deliverer then compares your signature to the publicly available handwriting sample, verifying that they are a match before delivering your letter. When your signature matches, the letter is considered to be DKIM aligned.

If you drop off a letter without signing it, the postal clerk instead signs the envelope with her own signature that matches her broadcasted handwriting sample. Just like we saw with SPF, a bad actor could exploit this loophole by dropping off an unsigned letter at your local post office. Because the postal clerk's signature and handwriting sample are a match, the letter will pass DKIM. However, it won't be in alignment because it's not your signature. 

##A passing grade is not good enough

This illustrates why getting a passing grade on SPF and/or DKIM is not enough —one of them has to be in alignment. Without alignment, your customer's email provider can't verify you are who you say you are. And if they refuse to deliver unauthenticated messages, like Google and Yahoo are, your message will never make it to your customer's mailbox.

DKIM alignment is the easier of the two protocols to implement. To implement DKIM alignment, check your Email Service Provider's documentation. Most ESPs have instructions that will walk you through the process of authenticating your domain and configuring DKIM. 
