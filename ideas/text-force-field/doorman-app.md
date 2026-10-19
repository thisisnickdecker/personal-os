# Safe Text: The Doorman App (v2 concept)

*2026-10-19. Follow-up to research.md.*

## The idea
Instead of a filter, build an app you use *in place of* iMessage. It works like Opal, but for texts. Every message waits at the door. The Doorman lets in contacts and real people, holds codes for you, and turns away spam into a folder you review at set times. Turn off Messages notifications on your phone and let the Doorman decide what buzzes you.

Folders:
- **Let in**: contacts, plus strangers who seem real ("you may have met this person")
- **Codes & deliveries**: verification codes and shipping updates; buzz you right away, then tidy themselves up
- **At the door**: not urgent, review at your set times
- **Turned away**: political, sales, real estate, scams; shown in the nightly email; one tap to mark safe or delete

## Can it be done?
- **An iPhone-only app: no.** Apple doesn't let any App Store app read or send your iMessages/SMS. That's why "Opal for iMessage" doesn't exist as an iPhone app.
- **With a Mac in the loop: yes.** A Mac signed into your Apple ID (with Text Message Forwarding on) stores every iMessage and SMS in a local database (`~/Library/Messages/chat.db`). A program with Full Disk Access can read it and send replies through Messages via AppleScript. This is how BlueBubbles, Beeper's iMessage support, and the `imsg` CLI all work.

## Architecture (personal MVP)
```
iPhone ──(Apple ID sync)──▶ Mac mini (always on, at home)
                              └─ doorman.py
                                  1. read new rows from chat.db every few seconds
                                  2. contact? → Let in
                                     code/delivery? → Codes
                                     else → Claude decides: let in / at the door / turned away
                                  3. push notification ONLY for Let in + Codes (Pushover or ntfy)
                                  4. serve a small web app (Safe Text) over Tailscale
                                  5. 8pm: email the "turned away" list
iPhone home screen: Safe Text web app (folders, mark safe, delete, reply)
iPhone settings: Messages notifications OFF
```

## Limits to know up front
- The Mac has to stay on (a Mac mini is ideal).
- "Delete" and "mark read" live in the Doorman's own records. They don't clean up the real Messages app, which can only be done with the BlueBubbles Private API and reduced Mac security. Skip that for the MVP.
- Messages notifications are off, so if the Mac or the Doorman goes down you miss alerts. Add a "Doorman is offline" ping.
- Your texts pass through Claude's API. That's fine for you; other users would need a clear privacy story.

## Prior art
- [BlueBubbles](https://bluebubbles.app/): open-source Mac server that relays iMessage to other devices. No filtering.
- [Beeper](https://www.beeper.com/): universal inbox. iMessage works on Mac only, and the connection keeps breaking as Apple blocks it.
- [imsg CLI](https://github.com/steipete/imsg): reads chat.db and sends via AppleScript. The building block.
- [RPLY](https://9to5mac.com/2025/02/06/new-mac-app-rply-offers-an-ai-fix-for-imessages-you-forgot-about/): Mac app that uses AI to surface texts you forgot to answer. About $30/mo. Not a filter.
- [Caspian](https://www.heynox.com/): Mac app that pulls together what needs your attention across Gmail, iMessage and more.
- Nobody combines a doorman, review times and a funny nightly report. That's the gap.

## Build plan
- **Tonight, free:** iOS Settings → Apps → Messages → Screen Unknown Senders. iOS 26 already shows verification codes from strangers in your main list for an hour.
- **Weekend 1:** `doorman.py` on the Mac reads chat.db, sorts with Claude, sends the nightly email.
- **Weekend 2:** Safe Text web app (folders, mark safe, delete, reply) + push notifications for Let in. Then turn off Messages notifications on the phone.
- **Weeks 3–4:** live with it. Success = your phone only buzzes for people and codes.
