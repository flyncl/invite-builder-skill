# Invite-Builder Skill for Claude

Build a self-designed HTML email invitation that lands in the inbox (not spam), with one-tap RSVP buttons, an add-to-calendar link and .ics file, a header visual, a test email to the host first, and RSVP tracking in a spreadsheet. Never emails anyone without explicit approval.

## How to install

1. Unzip this file.
2. Copy the `invite-builder` folder into your Claude skills directory:
   - **Claude Code:** `~/.claude/skills/` (macOS/Linux) — then restart Claude Code, or run `/skills` to confirm it appears.
   - **Claude.ai (Pro/Team):** upload the folder as a project skill, or paste `SKILL.md` into a new skill.
3. Start a chat and say something like: "Use the invite-builder skill to make an invitation for my dinner party on December 19."

## What's inside

- `SKILL.md` — the full skill: event details, guest-list tracker, header visual, email build rules (learned the hard way), RSVP mailto buttons, calendar link + .ics, text-message version, preview and test-send, sending, and ongoing RSVP tracking.

Made by Clare (@toomanyclicks).
