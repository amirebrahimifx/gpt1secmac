# gpt1secmac
This is a script which, when you make an Apple shortcut which has a keyboard shortcut set, then you can access ChatGPT on almost all apps on Mac and ask questions as fast as possible with a lot of context for that AI (screenshot of that specific window you are working on). It's only available for Mac.



Text+Image Quick Ask (AppleScript)

Capture the frontmost app’s front window as an image, bundle it with an optional text excerpt from your clipboard, then pop open ChatGPT Quick Ask and paste both—automatically.

Works great as a one-key “ask about what I’m looking at” helper.

⸻

What it does (TL;DR)

When you run the script:
	1.	Reads any text already on your clipboard (optional “excerpt”).
	2.	If the tiny Quick Ask panel is currently frontmost, it briefly hides it so the correct app is captured.
	3.	Finds the frontmost window’s position/size (via Accessibility).
	4.	Pixel-correct captures just that window to the clipboard (screencapture -R …).
	5.	Builds a prompt: your trimmed excerpt (if any) + a short instruction string.
	6.	Re-opens Quick Ask (⌥Space), focuses its input without switching Spaces.
	7.	Pastes the image first, then the text (safer across inputs).
(You can flip to single-paste mode if you prefer.)
	8.	Optionally presses Return to send (autoSend).
	9.	Clears your clipboard after a few seconds (autoClearSeconds) so you don’t keep a giant PNG around.

⸻

Requirements
	•	macOS 10.15+ (uses Screen Recording TCC) — Ventura/Sonoma/Sequoia are fine.
	•	ChatGPT for macOS installed and Quick Ask enabled (default hotkey: ⌥Space).
	•	No third-party packages required.

⸻

Setup: pick one

Option A — Run inside Shortcuts (fastest for iterating)

Use Shortcuts’ AppleScript action (AppleScript Runner) and paste the script as-is.
	1.	Open Shortcuts → New Shortcut (e.g., “Quick Ask: Text+Image”).
	2.	Add Run AppleScript (or “AppleScript” / “AppleScript Runner”).
	3.	Paste the script.
	4.	(Recommended) Assign a keyboard shortcut from the Shortcut’s ⓘ panel.

Permissions you must grant (one-time):
	•	Accessibility → enable Shortcuts Events.app
Path: /System/Library/CoreServices/Shortcuts Events.app
	•	Screen Recording → allow Shortcuts Events on first capture or add it manually.
	•	Automation → when prompted, allow Shortcuts/Shortcuts Events to control:
	•	System Events
	•	ChatGPT

Why “Shortcuts Events”? It’s the background helper that actually runs your AppleScript from Shortcuts. It—not the visible Shortcuts window—needs the permissions.

Option B — Package as a tiny app (cleanest long-term)

Export the script from Script Editor as an application and (optionally) launch it from Shortcuts or a launcher.
	1.	Open Script Editor, paste the script.
	2.	File → Export…
	•	File Format: Application
	•	Uncheck “Stay open after run handler”
	•	Save as /Applications/Kos.app (or any name).
	3.	(Optional, prevents focus steal) mark it as a background agent:

/usr/libexec/PlistBuddy -c 'Add :LSUIElement bool true' "/Applications/Kos.app/Contents/Info.plist" 2>/dev/null || \
/usr/libexec/PlistBuddy -c 'Set :LSUIElement true' "/Applications/Kos.app/Contents/Info.plist"
codesign --force --deep --sign - "/Applications/Kos.app"


	4.	To trigger from Shortcuts, add a Run Shell Script action:

/usr/bin/open -gj "/Applications/Kos.app"



Permissions you must grant (one-time):
	•	Accessibility → enable your app (e.g., Kos).
	•	Screen Recording → enable your app on first capture.
	•	Automation → allow your app to control System Events and ChatGPT when prompted.

⸻

First-run checklist (both options)
	•	Open System Settings → Privacy & Security and review:
	•	Accessibility: ON for Shortcuts Events (Option A) or your app (Option B).
	•	Screen Recording: allowed for the same runner.
	•	Automation: under your runner, allow control of System Events and ChatGPT.
	•	Optional macOS tweak to avoid Space-hopping:
	•	Desktop & Dock → Mission Control → turn OFF
“When switching to an application, switch to a Space with open windows.”

⸻

How to use
	•	Put the app/window you care about frontmost.
	•	Hit your Shortcut’s hotkey.
The tool grabs that window, opens Quick Ask on your current Space, pastes the screenshot + text, and (optionally) sends.

Tip: the script ships in two-paste mode (image then text). It’s more reliable in chat inputs.

⸻

Configuration flags (top of script)
	•	autoSend — false by default. Set true to press Return after pasting.
	•	twoPasteMode — true (paste image, then text). Set false for a single mixed paste.
	•	includeExcerpt — true (include clipboard text you had before running).
	•	maxExcerptChars — default 800 (long clipboards get trimmed).
	•	autoClearSeconds — default 5 (clears the clipboard after run; set 0 to disable).

⸻

What to expect the first time (prompts)

You’ll see up to three system prompts—the runner is the one requesting:
	•	Screen Recording (for screencapture).
	•	Accessibility (for reading window geometry + sending keystrokes).
	•	Automation (controlling System Events and ChatGPT).

If you see a prompt naming some other app (Notes/Notion/etc.), that means you launched from a path that attributes the request to that app. Prefer Option A/B above to keep permissions tied to the correct runner.

⸻

Troubleshooting
	•	“Shortcuts is not allowed assistive access” (-25211)
Add Shortcuts Events.app to Accessibility and Screen Recording.
	•	Prompt says “<random app> wants to control your computer”
You ran from that app’s context. Deny is sticky. Either run via Option A/B or reset that app’s Accessibility entry:

tccutil reset Accessibility <bundle id>
# e.g. com.apple.Notes


	•	It jumps to another Desktop/Space
Make sure your copy includes the no-activate helper (focus without frontmost true) and turn off the Mission Control setting noted above.
	•	Shortcuts hangs
Don’t use open -W. If you build a “restart” Shortcut, add a timeout loop or just fire-and-forget:

/usr/bin/open -gj "/Applications/Kos.app"


	•	Codesign / “resource fork” errors (Option B)

xattr -rc "/Applications/Kos.app" && codesign --force --deep --sign - "/Applications/Kos.app"



⸻

Security notes
	•	The script only reads your clipboard and captures the front window to your clipboard; it doesn’t upload anything by itself. You choose to paste into ChatGPT Quick Ask.

⸻

Pro tips
	•	Keep one runner per setup (either Shortcuts-embedded or the app). Mixing runners causes duplicate prompts.
	•	Wrap the UI block in a small with timeout to avoid rare hangs.
	•	If you iterate a lot, Option A (Shortcuts) is perfect; once stable, consider Option B so permissions live with a tiny app.

⸻

If you want a sample Shortcut or a pre-built app bundle, add a Releases section and I’ll outline packaging steps.
