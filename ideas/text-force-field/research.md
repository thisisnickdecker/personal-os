# Text Force Field: Idea Validation

*Research date: 2026-10-19. Source: morning voice memo idea dump.*

## The idea in one line

A buffer between your phone number and your attention. Every text from a stranger gets screened, junk (political, sales, "we want to buy your house", scams) disappears silently, and once a day you get a short note: *"Hey Nick, I kept 15 people out today. Here's who, in case I got one wrong."*

---

## 1. Is it technically possible?

**Yes, with one real catch on iPhone.**

### iPhone (Apple's SMS filter extension)

Apple has offered a supported way to do this since iOS 11: the **Message Filter Extension** (IdentityLookup framework). Every SMS filter app on the App Store uses it.

What it can do:
- iOS runs your filter on every SMS/MMS from an **unknown sender** (anyone not in Contacts) and asks it to classify the message.
- It can sort messages into Allow, Junk, Promotions or Transactions (2FA codes, deliveries). Filtered messages land in a separate folder and **don't notify you**. That is the force field.
- It can **send the message to your own server** ("network deferral"), so an LLM can make the call instead of just keyword rules.

What it can't do (this shapes the product):
- **Contacts are never filtered.** That's fine, since it matches your "is it from my contacts?" rule anyway.
- **iMessage isn't covered.** Most spam arrives as SMS, but some comes over iMessage, and Apple's own filters handle those.
- **Only one filter app can be active at a time.** You'd be replacing SpamHound, Robokiller and the like, not stacking on top of them.
- **The extension is sandboxed for privacy.** It has no network access of its own, and per Apple's WWDC guidance it can't write to storage shared with its main app. The server request built by iOS contains only `sender` + `message text` + `app version`, with **no user ID and no recipient number** ([Apple forum thread](https://developer.apple.com/forums/thread/110884)).

**What that means for the daily digest:** Apple deliberately makes it hard to build a per-user "here's everything I blocked" report.
- **For you as the guinea pig, this doesn't matter.** If you're the only user, every request hitting your server is yours, so a server-side digest just works. 🎉
- **For a multi-user iPhone product**, the server can't tell users apart. Realistic options:
  1. Make the digest an **in-app view** built from the filtered folder that iOS already keeps, plus a local notification, rather than an email.
  2. Use a **"shield number" model**: give each user a second number (via Twilio or similar) to hand out to forms, campaigns and businesses. Their texts are forwarded or summarized, and a digest is fully possible. This works on any phone but doesn't protect the number you already have.
  3. **Android first** (see below).
- Reported quirk in iOS 26: once a thread has been moved to Spam, later messages in it may skip the extension ([forum report](https://developer.apple.com/forums/thread/841074)).

### Android

Android allows much more. An app can become the **default SMS app** and see, sort and log every message, so a full digest with no workaround is possible. It's more work, though, because you're building a whole messaging app. Google is also moving hard into this space: Google Messages now has on-device scam detection on by default for unknown senders, and is adding cloud Gemini checks ([Droid Life, Oct 2026](https://www.droid-life.com/2026/10/14/google-messages-is-getting-a-massive-gemini-powered-scam-defense-upgrade/), [BleepingComputer](https://www.bleepingcomputer.com/news/security/android-gets-ai-defenses-against-messaging-scams-fake-calls/)).

### Why political texts get through everything else

Political texts are **exempt from the Do Not Call registry**, and "peer-to-peer" campaign texting (a volunteer clicks send on each message) falls outside the TCPA autodialer rules. Volume was about 15B messages in 2022, with about 25B projected for 2024, and 2026 midterm volume is surging ([Fox 13](https://www.fox13news.com/news/political-texts-surging-2026-here-how-reduce), [Tatango](https://www.tatango.com/blog/is-it-legal-to-send-political-text-messages/), [Hoodline](https://hoodline.com/2026/07/robo-text-blitz-hits-washington-voters-ahead-of-2026-midterms/)). Campaigns rotate numbers constantly, so blocklists lose. **Reading the content is what works**, which is exactly what an LLM is good at.

---

## 2. Does it already exist?

**Mostly, yes.** The "force field" itself is crowded. The *digest + personality* layer is the open part.

| Who | What | Gap vs. your idea |
|---|---|---|
| **Apple iOS 26 (free, built in)** | "Screen Unknown Senders" + "Filter Spam": strangers go to a separate list with no notifications ([Tom's Guide](https://www.tomsguide.com/phones/iphones/how-to-screen-texts-from-unknown-senders-in-ios-26-messages), [iPhone Life](https://iphonelife.com/content/how-to-filter-unknown-known-senders-messages-your-iphone)) | Blunt: *all* strangers get muted, including your doctor and the plumber. No summary. |
| **Google Messages (Android, free)** | On-device AI scam detection, plus opt-in cloud Gemini | Focused on scams, not "annoying". No digest. |
| **ElectionDeflection** | Political-text filter, AI in the Pro tier ([site](https://electiondeflection.com/)) | Political only |
| **PolitiShield**, **Stop Political Spam Texts** (open source) | Keyword/rule political filters ([App Store](https://apps.apple.com/us/app/politishield/id6746817630), [App Store](https://apps.apple.com/us/app/stop-political-spam-texts/id6782703267)) | Rules, not understanding |
| **Junkman**, **Text Spam Block** | On-device ML spam filters ([App Store](https://apps.apple.com/us/app/junkman-ai-spam-sms-blocker/id1591815272), [App Store](https://apps.apple.com/us/app/text-spam-block/id6499238158)) | No digest |
| **Robokiller** | Calls + texts, about $40/yr, weekly stats ([WhistleOut](https://www.whistleout.com/CellPhones/Apps/best-spam-blocker-apps)) | Mixed reviews, call-focused, no daily "here's what I caught" |
| **Truecaller** | Caller ID + spam, about $75/yr premium | Privacy concerns, call-focused |
| **Carriers** (T-Mobile Scam Shield, Verizon Call Filter, AT&T ActiveArmor) | Network-level spam blocking | Doesn't catch P2P political or "legit" junk |

**Honest read:** blocking is commoditizing fast, and Apple and Google are doing it for free. What nobody does well:
1. **Judgment on unknowns.** Let the dentist, the delivery driver and the new friend through, and block the campaign and the land-flipper. iOS's "mute all strangers" is too blunt and keyword apps are too dumb. An LLM with *your* preferences ("I care about local school stuff, not national campaigns") is a real step up.
2. **The daily ritual.** *"I blocked 15 people today: 9 political, 3 real-estate, 2 scams, 1 I'm not sure about 👀."* That turns an invisible feature into a felt one. Nobody owns this, and it's the brand.
3. **A voice and a vibe.** Spam apps are all fear-based ("SCAM ALERT!"). Yours is calm and a little funny (force field, inner sanctum, condom joke), which stands out.

---

## 3. How to build it

### Phase 0: Baseline (this week, $0, about 10 minutes)
Before building anything, measure the pain.
- Turn on iOS 26 **Settings → Apps → Messages → Screen Unknown Senders + Filter Spam**.
- For 7 days, once a night, open the Unknown Senders / Spam lists and tally: total, political, sales/real estate, scam, **"actually wanted that"** (that last one is the key number).
- If iOS alone feels like 90% of the value, that's a real finding too. Then the product is mostly the digest and the smarter judgment.

### Phase 1: Personal MVP, you as guinea pig (about 1–2 weekends)

```
Incoming SMS from stranger
        │
        ▼
iOS Message Filter Extension ──(network deferral: sender + text)──▶ Your server
        ▲                                                             │
        │                                         1. quick rules (short codes, "STOP to opt out", known 2FA)
        │                                         2. Claude (small/fast model) classifies:
        │                                            allow / political / sales / real-estate / scam / unsure
        │                                         3. log to DB
        └────────────── allow | junk | promotion ◀────────────────────┘

Every evening at 8pm: cron → read today's log → Claude writes a 3-line summary → email/text to you
```

**Stack (cheap and vibe-codeable):**
- **iOS app:** a tiny SwiftUI shell + Message Filter Extension (about 100 lines of Swift). Needs an Apple Developer account ($99/yr), an associated domain for the filter server, and TestFlight to install on your own phone. Claude Code can write nearly all of it.
- **Server:** one endpoint on Vercel (you already deploy there) or Cloudflare Workers. Postgres/SQLite (Supabase, Neon or Turso) for the log.
- **Classifier:** Claude's smallest/fastest model with a short system prompt that includes your preferences. The cost is fractions of a cent per message, so even 50 spam texts a day is pocket change.
- **Digest:** Vercel Cron + Resend (email) or Twilio (text) once a day.
- **Safety rail:** if the server is slow or down, default to **allow**. Never lose a real message.

**Success criteria after 2 weeks of living with it:**
- Fewer than 1 "real message wrongly blocked" per week
- You actually *read* and *enjoy* the digest
- Noticeably fewer interruptions (count notifications before and after)

### Phase 2: Friends-and-family test (weeks 3–6)
- Text 20–30 people a one-liner: *"If I built a force field for your phone that silently kills political/spam texts and sends you a funny nightly 'here's who I blocked' report, would you use it? Would you pay $2/mo?"* Count the yeses, and **pay attention to who asks "when can I have it?"**
- Give 5–10 iPhone friends TestFlight access. Because the iOS digest can't be per-user (see §1), use the in-app view + nightly local notification for them, or try the shield-number variant.
- Watch the numbers: still enabled after 2 weeks? How many false positives?

### Phase 3: Decide
- **Strong pull** (people nag you for it, keep it on, would pay) → App Store launch, $2–3/mo or $20/yr, with the digest as the hero feature. Launch into peak political-text season (a 2026 midterms story, or the 2028 primaries).
- **Lukewarm** → keep it as your personal tool. That's still a win.

---

## 4. Naming

Two lanes you named, plus a third:

**The prophylactic lane (funny, memorable, risky with Apple's App Store review + ads):**
- **Textdom**: a pun on condom, short, sticky
- **Rubber**: "wrap your number"
- **Safe Text**: practice safe text 😏 (taglines write themselves)

**The force field lane (aspirational, brandable):**
- **Forcefield**: the obvious winner if the name/domain is available
- **Sanctum**: "protect your inner sanctum"
- **Moat**
- **Ward**: as in a protective ward

**The bouncer lane (a personality to talk to):**
- **Bouncer** / **Doorman**: "Your doorman turned away 15 people today."
- **Velvet Rope**

My pick: a **force field or doorman name for the product, with the condom joke in the marketing** ("Safe text. Every time."). App Store review and ad platforms get squeamish about sexual puns in the app name, but a tagline is fine. The doorman framing also gives the daily digest a natural voice.

*Domain availability wasn't checked (the research sandbox couldn't reach registrars). Check `.com`/`.app` for the shortlist on Namecheap or Porkbun before falling in love.*

---

## 5. Risks and open questions

- **Platform risk:** Apple and Google are shipping this for free and improving every year. The moat is judgment + digest + brand, not blocking.
- **Privacy:** you'd be sending strangers' texts to a server and an LLM. That's fine for you. For other users it needs a clear privacy policy, no retention beyond the digest window, and ideally on-device classification later.
- **False positives are fatal.** One missed doctor's or school message erodes trust. Bias toward "unsure → allow and flag in the digest".
- **iMessage spam** isn't reachable by third-party filters.
- **iOS digest for multiple users** needs one of the workarounds in §1. Decide on it before promising a "nightly email" to anyone else.

---

## 6. Next 3 actions

1. **Today:** turn on iOS Screen Unknown Senders + Filter Spam and start the 7-day tally (Phase 0).
2. **This week:** send the one-line "would you use this?" text to 20 people. Pick a working name.
3. **Next weekend:** with Claude Code, build the Phase 1 filter extension + classifier + nightly digest for your own phone.
