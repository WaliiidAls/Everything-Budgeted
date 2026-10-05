

## Quick logging

- **One line:** type `4.50 coffee`, `12 uber #transport` or `30 lunch yesterday` in the box at the top and press Enter. The tag is guessed from what you logged before (on your device); type `#tag` to pick one.
- **One tap:** your most frequent spends from the last 45 days show as chips under the box.
- **Apple Pay (iPhone):** in Shortcuts → Automation → **Transaction**, pick your cards, then add *Open URL*:
  `https://waliiidals.github.io/Everything-Budgeted/?amount=[Amount]&merchant=[Merchant]&ref=[Current Date]`
  Every tap-to-pay purchase opens the app and logs itself. The `ref` stops the same purchase being logged twice.
- **Other automations:** `?q=4.50 coffee` logs a quick-log line; `?quick=1` opens the app with the quick-log box focused (also the "Quick log" home-screen shortcut).

## Duplicate detection

Before any spend is saved (quick log, chip, checklist tick, analytics form, automation) it is compared with spends on the same day with the same amount. It is flagged when enough extra evidence lines up: logged within minutes of each other, the same or similar note/merchant, the same tag, or coming from two different routes (e.g. Apple Pay automation and by hand). You get an "Already logged?" prompt with *Don't add* / *Add anyway*, so two genuine coffees are never blocked. A repeated automation `ref` is skipped automatically.
