---
title: 'Migrating Away from G Suite Legacy Free Edition: 2022 Email Providers Overview'
date: '2022-02-08T01:04:22-05:00'
author: Adrien Poupa
url: /migrating-away-from-g-suite-legacy-free-edition-2022-email-providers-overview/
---

On January, 19 2022 Google [announced](https://9to5google.com/2022/01/19/g-suite-legacy-free-edition/) they were pulling the plug on the G Suite Legacy Free Edition, that was offered from 2006 to 2012. Among other things, it was offering Gmail with a custom domain.

Like many others, I subscribed to what was called Google Apps back then, to give custom e-mail addresses to my family (ie: surname@lastname.fr). Over the years, we used those Google accounts like personal accounts and bought applications, used Google Drive, Google Documents, and so on.

I was disappointed by the decision but not surprised, given Google’s long-standing history of [killing its products](https://killedbygoogle.com/) and [going back on their word](https://www.bbc.com/news/technology-54919165). What surprised me more is the current lack of possibility to migrate to a free Google account (even though it might be coming given [Workplace Essentials](https://workspace.google.com/essentials/) announcement and the [form](https://forms.gle/KuiqW4GswCG3M1bp8) in their knowledge base), as well as the absence of a family-oriented offer. Google’s suggestion to pay $6/user/month is understandable for a business, but unreasonable for a family. Sweetening the deal by offering discounts for the first year won’t do either, sorry Google.

It also made me think about how much of my digital life is tied to a single vendor, and how easily I got trapped into it without realizing it. I have an Android phone, I receive my emails on Gmail, upload my photos to Google Photos, have an application on the Play Store, backup my data on Google Drive and use Google Docs frequently, all from my G Suite Legacy account. Yet all it takes is an automated [account suspension](https://news.ycombinator.com/item?id=24791357) to take that away, and talking to a human might not be an option. I realized that when Google took down one of my Play Store apps for seemingly no reason and I had to fight tooth and nail to get it back, choosing carefully each word before sending the form with no guarantee of success.

Therefore, because I want to keep my vanity email I used for the last decade, given the unlikeliness of Google letting me do so for free and because I don’t want to spend $216 yearly on email (6\*12\*3), I started to look for an alternative, with the added benefit of a forced de-googling 🙂

I wanted to be able to create at least 3 email addresses with a reasonable size (let’s say 3-5Gb minimum), be able to achieve good delivery with SPF, DKIM and DMARC, and have catchall support.

### Drop-in Replacements

[Zoho](https://www.zoho.com/mail/zohomail-pricing.html), founded in 1996, has an interesting $1/month/user plan. The webmail is clean and they offer a [coherent email suite](https://www.zoho.com/mail/mailsuite-apps.html). They have [support for catchall](https://www.zoho.com/mail/help/adminconsole/catch-all-setup.html). It’s a solid option, but I don’t like having to pay increasingly more as I add users. They have a free tier though, but I don’t consider it a viable option since it does not include any POP/IMAP access. They recently introduced [Zilium](https://www.zoho.com/zillum/) as a Workplace alternative for families, for those that need more than just emails, starting at $6/month for 3 users.

Surprisingly, Microsoft has a good family offer. [Microsoft 365 Family](https://www.microsoft.com/en-us/microsoft-365/p/microsoft-365-family/cfq7ttc0k5dm?activetab=pivot:overviewtab) will offer 6 seats for all the Office software that we know (and love?), and a whopping 1tb/user in OneDrive for $100/year. It gets cheaper with [Costco](https://www.costco.com/microsoft-365-family-15-month-subscription-(e-delivery).product.100462462.html), or if you benefit from the [Home Use Program](https://www.microsoft.com/en-us/home-use-program) with your employer. It advertises the use of custom domains with GoDaddy but it is [possible to use any registrar](https://www.reddit.com/r/Office365/comments/ft15pk/use_personalized_domain_with_outlook_and_office/). It is a good offer if you need the Office licenses, if you like the Outlook webmail, or if you could use the OneDrive space. Unfortunately catchall are [not supported without using tricks](https://ca.godaddy.com/help/catch-all-email-not-supported-with-microsoft-365-40130), and [neither are DKIM/DMARC](https://answers.microsoft.com/en-us/outlook_com/forum/all/microsoft-365-family-with-custom-domain-dkim-and/3b739764-93ea-4558-8161-732f09d34992).

[iCloud](https://support.apple.com/en-us/HT201238) also offers custom emails for $1/month. There is no catchall though and no DKIM/DMARC from iOS applications apparently. I guess it is still an OK option if you are in the Apple ecosystem or if you’re already paying for iCloud.

### Privacy-oriented Alternatives

Based in Switzerland, [ProtonMail](https://protonmail.com) is a good solution for privacy-minded folks thanks to their encryption, and the webmail is well thought. However, the prices are way too step for me at $6.25/month/user, which is more than what I would pay for Google Workspace. They have a cheaper tier for $4 monthly but it’s geared to single users. Interestingly, they apply a $1 = **€**1 = CHF 1 conversion rate, so I guess you’re better off paying in the Uncle Sam’s currency 🙂

[Tutanota](https://tutanota.com) is another privacy-oriented German alternative, offering encrypted emails as well for as low as **€**1/user/month. Their webmail is [open-sourced](https://github.com/tutao/tutanota), which is pretty cool. It only works online [for the moment](https://github.com/tutao/tutanota/issues/590), though.

Autralian-based [Fastmail](https://www.fastmail.com) markets itself as a a privacy-friendly Gmail alternative, starting at $2/month/user, and seems to be a HackerNews favorite.

In Germany, [Mailbox.org](https://mailbox.org/en/) offers solutions starting at €3/month/user for a custom domain, and they support encryption.

### Email Services

The following are taking email very seriously as it is their main business.

[Migadu](https://www.migadu.com/) is another reasonably-priced Swiss provider. Unlike the previous providers their pricing is based on usage rather than the number of users/mailbox. Their family plan starts at [$19/year](https://www.migadu.com/pricing/) for an [*almost* unlimited](https://www.migadu.com/pricing/#what-is-the-domains-limit-on-the-micro-plan) number of domains but they have very low “soft” limits. like 20 emails out per day for the whole account. You may cross that but they may not like it. They used to have a free plan that they retired at the beginning of the pandemic. They use [Rainloop](https://www.rainloop.net/) as webmail that does not look very modern in my opinion. While they had plans to move to a custom solution, it seems [very little progress](https://www.migadu.com/blog/redesign/#the-webmail) was made during the past year.

[MXRoute](https://mxroute.com/) is another no-nonsense email provider with a minimum flat fee of $45 for an almost unlimited plan. It seems to be a one-man operation with the owner being very active on LowEndTalk/Reddit. The documentation is kinda.. [baroque](https://mxroute.com/docs/#/General/GDPR) 🙂 but that may appeal to some people. Moreover, deliverability seems to be top notch due to using a random pool of IP to send emails, and shuffling through them when an email does not make it. They use Rainloop and Afterlogic as webmails. Afterlogic comes with a calendar, but if you need a solid calendar, [I would not use them](https://mxroute.com/docs/#/General/Do_You_Sell_Calendars). It is a good option for multi domain email if you do not really need support as they expect their users to know what they are doing (but if you are reading this I guess that you are!).

[Purelymail](https://purelymail.com/) is an even cheaper alternative focusing on email with a RoundCube webmail. It is by far the [cheapest](https://purelymail.com/pricing) of the bunch ($10/year with no limit, or even cheaper with [advanced pricing](https://purelymail.com/advancedpricing)). If you a bare bone provider is good for you and/or you’re only willing to pay a very low price this may be your best bet. Similarly to MXRoute, it is however managed by a single person, so beware of the bus factor.

[Postale](https://postale.io/) is a good alternative for either very cheap single use ($1/month for 1 domain and 2 addresses) or reasonably-priced multi domain ($5/month for unlimited domains and 25 addresses). Their webmail is a recent version of [Roundcube](https://roundcube.net/), which is not ideal in my opinion but still better than older versions.

### Domain Registrars + Email

Using the email service of a registrar can be convenient and really cheap. I’d argue it’s both a blessing and a curse, given they may care a little less about emails since it’s not their core business, but they may also be more inclined to lower their margins on emails because they’re making their money elsewhere.

Europe giant [OVH](https://us.ovhcloud.com/) has some interesting offers, and a complicated history for those. Historically, it offered MXPlans that were one time fees plans. Each domain registered with OVH grants you access to a free hosting offer, the [Start 10M hosting plan](https://docs.ovh.com/gb/en/hosting/activate-start10m/), that would include 1 free email address. If you needed more, You would pay **€**5 once for 5 addresses, for life, or more for more addresses. Those offers are really interesting on paper but come with a few major caveats: no catchall address, no DKIM/DMARC and you have to register your domain with them to access those offers. Moreover, depending if your offer was migrated or not, you would end up with either an ancient version of Roundcube, or Outlook Web App hosted by OVH. Also, OVH caused [quite a stir](https://community.ovh.com/t/arret-de-la-commercialisation-du-mx-plan/5578) in 2017 when it announced it would retire those plans and migrate them to a monthly paid plan (sounds familiar? :)) Overall it’s not necessarily a bad option but pretty much like everything else at OVH, it’s dirt cheap with very rough edges.

Switzerland based [Infomaniak](https://www.infomaniak.com/en) brands itself as an alternative to the web giants. It develops the [Infomaniak Suite](https://www.infomaniak.com/en/myksuite), an hybrid of in-house tools and open source solutions presented as alternative to Google Workspace. Their [email offer](https://www.infomaniak.com/en/hosting/service-mail/ecosystem) is affordable at **€**1.5/month for 5 addresses, and 1 address is included for free for every domain registered with them (I also found their prices to be significantly lower than OVH). I really liked the UI of their custom webmail, they support catchall, DKIM and DMARC. Unfortunately they have no mobile client but it is apparently worked on, and their offer is limited to domains your register with them (or you would have to order a DNS zone with them). If you are based in Switzerland, France, Belgium, Luxembourg, Monaco and the United States, you may create create a free address to see what it looks like.

[RackSpace](https://www.rackspace.com/email-hosting/webmail/pricing) is a decent professional alternative for those seeking a Microsoft environment with a smaller price tag.

[Gandi](https://www.gandi.net/en-US)‘s domain prices are a tad higher than the norm, but they come with 2 free email addresses and 10,000 aliases and forwarding addresses. Their webmail is [SoGo](https://www.sogo.nu/). It can be a good choice if you use them as registrar or intend to do so.

[Namecheap](https://www.namecheap.com/hosting/email/) offers professional emails for a reasonable cost but unfortunately no free address for a domain registered with them.

### How About Self-Hosting?

This one is interesting. I had been toying with the idea of self-hosting my emails as it’s a cool project to do, and it lifts all the arbitrary limitations most providers put on emails (inbox size, number of users, number of emails going out, etc).

The general [consensus](https://news.ycombinator.com/item?id=29672846) is that while receiving email is simple, sending them out *and* achieving good deliverability is [not trivial](https://utcc.utoronto.ca/~cks/space/blog/sysadmin/EmailServersNoLongerPractical). It’s not an impossible task, but it requires some monitoring and it’s not guaranteed that Google or Microsoft will not block your IP tomorrow, because why not.

The first step would be to get a VPS or a private server, because most residential IPs are banned by bigger providers and your ISP most likely blocks port 25. Then you need to make sure the IP you get is clean on a tool like [mxtoolbox](https://mxtoolbox.com/blacklists.aspx). For all providers, SPF must be configured, and for Gmail, DKIM/DMAC as well. Outlook has arbitrary blacklists that apparently blocks new IPs by default so you’d need to be whitelisted.

Given that email has been around for a really long time, I have no doubt you will find plenty of tutorials to set your server the old way with Postfix and Dovecot. But there are now simpler solutions that are more maintainable.

[Mailinabox](https://mailinabox.email/) is a very popular and very simple solution that will turn an Ubuntu Server in a mailing server with a single command. It is not dockerized, though. It installs an IMAP/SMTP server, Roundcube webmail, Nextcloud Contacts and a control panel.

For Docker afficionados, [Mailcow](https://mailcow.email/) is a similar solution but comes with the SoGo webmail. It is more configurable too.

[Mailu](https://mailu.io/) is a Docker-compose/k8s solution that seems a bit less plug&amp;play but more configurable. It is based on [Poste.io](https://poste.io/).

Alternatively, [Docker-mailserver](https://github.com/docker-mailserver/docker-mailserver) is a more barebone solution with no admin panel but a lot of configuration and smaller requirements.

An interesting middleground for self hosted emails could be to handle reception with one of those solutions, but delay sending to external services to avoid the blacklist headache. Amazon SES is generally recommended for their low rates ($0.10 for every 1,000 emails). It should be as easy as changing the relay server in the configuration of those software.

### In Conclusion…

This article does not aim to present an exhaustive list of email providers, but rather to highlight the different possibilities GSuite Legacy users have. I remember when Google, Microsoft and Zoho would let us host our custom domains for free. It was good while it lasted.

But since that’s no longer the case, it’s a matter of what we’re willing to spend, whether that’s time or money. Personally I consider email to be critical so I don’t want to bother self-hosting it. Yes, it is quite easy technically with the aforementioned scripts and even the sending issues can be mitigated, but I don’t want to wonder if my backup solution is good enough, if the email I sent was actually delivered or what would happen if my server goes down or dies (OVH, remember? :)). I would rather offload this risk to people who do that for a living.

Initially, given I have registered my domains at OVH I wanted to use their MXPlan solution but the lack of catchall/DKIM/DMARC really put me off. Then I realized I wanted a clean webmail, and unfortunately I was not convinced by FOSS alternatives.

For the moment, I will try Infomaniak given their lower domain registration prices offset the email service cost, and I was really sold on their UI and philosophy. I have heard their support was good too, hopefully I will not need it.

Have I missed an amazing provider that you use? Do you disagree with what I have said? Let me know below, and happy migration 🙂