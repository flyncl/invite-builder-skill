---
name: invite-builder
description: Build a self-designed HTML email invitation that lands in the inbox (not spam), with one-tap RSVP buttons, an add-to-calendar link and .ics file, a header visual, a test email to the host first, and RSVP tracking in a spreadsheet. Use when the user wants to invite guests to an event by email, with or without a text-message version.
---

# The Invite-Builder Skill

## What it does

This skill was designed for Claude and works well in that environment. Makes a self-designed HTML email invitation that lands in your invitee's inbox (without getting diverted to spam), with RSVP buttons guests tap once, an add-to-calendar link, and a header visual. Ends with a test email to the user. Never emails anyone without explicit approval. Tracks RSVPs in a spreadsheet.

## Step 1 - Event details (ask first)

Ask these one at a time, in this order: one question per message (or per AskUserQuestion call), and wait for the answer before asking the next. Skip any the user has already answered. Collect:

- Event name and the exact wording they want (if they have it, use it verbatim)
- Date and time zone
- Start time AND end time. Always ask for both, as a separate question. Never assume or invent an end time: the add-to-calendar link and the .ics file both need it. If the host has no fixed end ("till late"), ask what end time to put in the calendar entry, and whether to show the end time on the invite itself or leave it as calendar-only.
- Full address (street, town, state, zip - the zip is needed for map apps)
- Hosts' names and sign-off (e.g. "XOXO Clare and Conrad")
- Any feature to highlight (band, speaker, dress code)
- RSVP-by date, and whether they want a "Maybe - remind me" option
- Which email account it sends from (a real personal inbox is best for deliverability)
- Whether they want a text-message version too, for guests who only have a phone number or who answer texts faster than email (see Step 3b)

Check the date and weekday match (e.g. confirm 19 December 2026 is a Saturday) and say so.

## Step 1b - Guest list

Ask: "Do you have a guest list to upload, or would you like me to start a spreadsheet for you?"

**A. They have one** (Google Sheet link, CSV or Excel upload, a past Paperless Post / Evite export, or a pasted list):

- Read it and build a clean tracker tab next to the original (never edit their original tab): ID (G001...), Guest name, Email, Phone, Invited (number), Status, Attending, Guest note, Previous response (if the source has one), Flag, Invite sent, Reminder sent, Last updated.
- Merge duplicates (same household under two addresses, a partner who replied separately, "Additional guest" rows) and note each merge in Flag.
- Set Status to "Not sent", or "Needs email" for phone-only guests and previously bounced addresses. Flag work addresses, which can filter invitations.
- Add a Summary tab: headcount (SUMPRODUCT of Attending where Status = Yes), counts of Yes / Maybe / No / Invited / Not sent / Needs email / Bounced, total households, max possible guests. Add a Status dropdown.
- Report back the count, the merges, and the list of guests missing an email address.

**B. They want a new one:**

- Create an xls file named "[Event name] [Year] Guest List" with the same tracker and Summary tabs, empty, and share the link.
- Tell them they can paste names and emails in, or give them to you in chat and you will add them.

**C. Not yet:** carry on with the design and come back to the list before Step 5. Never send without a list.

## Step 2 - Header visual

Ask: "Do you have a header image you'd like to use?"

**A. They upload one.**

- Look at it. Pick 2-3 colors sampled from it (the dominant accent, a dark for text, a light for background) and a font pairing that matches its mood (playful image -> rounded serif; photographic or sleek -> clean sans; vintage -> engraved serif).
- If it has a transparent background, flatten onto the invite's background color (white for white layouts). A transparent PNG on a white page looks cleaner than a boxed image.
- Build ONE design in keeping with it, then offer two variations only if they ask.

**B. They don't have one.**

- Ask what vibe they want (e.g. cozy, glam, retro, minimal, festive, elegant, silly) and any colors to use or avoid.
- Create the header image yourself:
  - Prefer an image generation tool if one is connected (e.g. Canva generate-image). Ask for a wide banner (2:1), the subject on a plain or transparent background, no text unless the user wants it.
  - Otherwise draw an original illustration as SVG and render it to PNG with Playwright.
  - Never reproduce a known character, logo, artwork or brand.
- Produce 3 distinct options, each with its own header, font pairing and palette (e.g. Playful, Modern, Classic). Show all three side by side at phone width and let the user pick or mix.

## Step 3 - Build the email (rules learned the hard way)

Email clients are not browsers. Follow all of these:

- Layout: nested `<table role="presentation">`, max width 520-560px, all styles inline. No CSS classes, no flexbox, no external images.
- Fonts: web-safe stacks only: 'Helvetica Neue',Helvetica,Arial,sans-serif, Palatino,'Palatino Linotype','Book Antiqua',serif, 'Times New Roman',Times,serif, Georgia,serif. Get personality from weight, letter-spacing, capitals and size, not from custom fonts.
- Do not put text on a colored background block. Some clients strip table backgrounds, leaving white text on white. Use colored TEXT on the page background instead. For a solid button, set the color both as bgcolor="#hex" on a wrapping `<td>` and as background: on the link.
- Header image: attach inline and reference as src="cid:att0" (first attachment). Resize to ~400-520px wide, JPEG quality ~45-60, target under 30KB so the base64 fits in a tool call. Set explicit width/height attributes.
- Left-align or center consistently - every section (details, feature, RSVP, sign-off) on the same edge.
- No em dashes in any copy. Use a hyphen.
- Use symbols that render everywhere (e.g. &#10052; snowflake, &#9670; diamond, &#128197; calendar). Avoid literal asterisks as decoration.

### RSVP buttons (mailto, no form or login)

Three buttons, each a mailto to the host address with a pre-filled subject and body:

- **Yes:** subject `RSVP YES: <Event>`; body asks "How many of us are coming (please fill in):" with a blank - never assume a headcount.
- **Maybe - remind me:** subject `RSVP MAYBE: <Event>`; body "Hoping to make it - please remind me closer to the date."
- **No:** subject `RSVP NO: <Event>`. URL-encode subject and body (urllib.parse.quote). Note under the buttons only if the user wants it: guests can also just reply.

### Add to your calendar

A link with a calendar icon to Google Calendar's template URL: https://calendar.google.com/calendar/render?action=TEMPLATE&text=<title>&dates=YYYYMMDDTHHMMSS/YYYYMMDDTHHMMSS&ctz=<IANA zone>&location=<full address>&details=<one line>

Also attach an .ics file (METHOD:PUBLISH, VTIMEZONE block, DTSTART/DTEND with TZID, LOCATION with escaped commas, CRLF line endings) for Apple Calendar and Outlook users.

Explain to the user that a link cannot push a calendar invite; a real calendar invite would expose the guest list to everyone.

## Step 3b - Text-message version (only if the user said yes in Step 1)

Write a short plain-text version the host can send from their own phone:

- Under about 320 characters if possible, so it does not split into several messages. Plain text only: no HTML, no images, no attachments.
- In the host's voice, with the same wording as the email: event name, weekday and date, start time, full address, the highlight (band, dress code), the sign-off.
- RSVP line: "Reply YES (and how many of you), MAYBE or NO". Do not use mailto links: they are clumsy on phones.
- Optional add-to-calendar line: the Google Calendar link from Step 3, only if the user wants it (it is long and makes the text look like spam).
- No em dashes. Emoji only if the user asks for them. Show it in a message-compose card so the user can copy it or open it in Messages. Offer a one-line reminder text too, for Maybe and no-reply guests.
- Tell the user plainly: Claude cannot send texts or read text replies. They send the text themselves, and either update the tracker when replies come in or tell Claude and Claude updates it. In the tracker, mark texted guests with "Invited (text)" in Status and the date in Invite sent, so the email run never sends them a duplicate.

## Step 4 - Preview, then test-send

- Render the HTML with the header swapped to a local file and screenshot with Playwright at 400px wide (device_scale_factor 2) and 640px. Look at it yourself before showing it. Save the preview as [YY MM DD] [Event] Invite Preview.png and deliver it.
- Iterate on wording, layout and colors from the user's feedback; re-preview each time.
- When they say test it: send the email to the user only, subject prefixed [TEST] (then [TEST 2], etc.), with the inline image and the .ics attached, plus a plain-text alternative body. Ask them to check: image shows inline, buttons open the pre-filled reply, the solid button is visible, the calendar link and file both work, it looks right on a phone.
- Save the final template, the .ics and the event facts to the Claude project if one is attached, so a later session can send it.

## Step 5 - Sending to guests (only when the user says they're ready)

- Use the tracker from Step 1b. Every row with an email and Status "Not sent" goes on the send; rows marked "Needs email" are listed for the user to resolve.
- Default: one Gmail DRAFT addressed to the user with all guests in BCC (the inline image must be attached per message, so personalised per-guest sends are not practical). The user approves or sends it themselves.
- Flag guests with phone numbers only, bounced addresses, duplicates, and work addresses that may filter invitations.
- After sending, mark those rows "Invited" with the sent date.

## Step 6 - Ongoing tracking (set up as a scheduled task)

Offer ongoing tracking against the same tracker: a scheduled task that can pick up newly added guests, read replies matching subject:RSVP subject:<Event>, update the sheet, draft (never send) reminders to Maybe and no-reply guests on the reminder date, and email the host an update only when something changed.

Before creating the task, ask these one at a time (one question per message or AskUserQuestion call, waiting for each answer):

1. Do you want me to monitor your inbox for RSVPs? Yes: Claude reads the RSVP replies (button replies and free-text replies to the invite), records each answer, headcount and note in the tracker, and emails you an update when something changes. No: Claude does not read the inbox, and the host updates the tracker themselves (or tells Claude in chat).
2. If yes: how often should I check your inbox for RSVPs? Options: Every hour (best in the last couple of weeks), Every few hours, Once a day (fewest update emails), or Other. Hourly is normally the shortest interval a scheduled task allows. If they want a different pace closer to the date (e.g. daily now, hourly in the final week), set the first schedule and offer a one-off reminder to change it.
3. Should new guests get their invite automatically when you add them to the tracker? Either Claude forwards the master invite to each new row (the host only types a name and email), or Claude creates a draft for the host to send, or neither. Automatic sending needs a Gmail send/forward tool in the scheduled run. If the organisation's settings remove it, say so up front and fall back to drafts.
4. If yes to question 3: how often should I check the tracker for new guests? Offer the same options as question 2, and suggest the same schedule as the RSVP checks so one task does both. Use separate schedules only if the host asks for different paces.
5. If they did not answer the text question in Step 1, ask now: Do you want a text-message version as well? If yes, do Step 3b.

If the answer to both question 1 and question 3 is no, do not create a scheduled task. Say that tracking is off and they can ask for it later.

Otherwise write the standing instructions to the Claude project (one .md per event), covering only what the host turned on, and create the scheduled task with the chosen schedule. Also tell the host which approval setting the task got.
