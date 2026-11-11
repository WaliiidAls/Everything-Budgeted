# Everything Budgeted



Every purchase SMS from your bank is sent to the app's link. The app reads the amount, the shop and the date, picks the shop's usual tag, and logs it. It happens in your browser; no server is involved.

### The link

```
https://waliiidals.github.io/Everything-Budgeted/?sms=<the SMS text, URL-encoded>
```

Also works for other automations: `?amount=3.5&merchant=Ananas&tag=Coffee&date=2026-11-11&ref=<unique id>`.

### Set up the automation (about 2 minutes)

1. Open **Shortcuts** → **Automation** tab → **+** (or *New Automation*).
2. Pick **Message**.
3. **Sender**: tap *Choose* and pick your bank's SMS sender (e.g. *Boubyan*). If it isn't a contact, type the sender name exactly as it appears in Messages.
4. **Message Contains**: type `KWD` (or whatever your purchase SMS always contains, e.g. `KD` or `د.ك`). This keeps OTPs and promos out; the app skips them anyway.
5. Choose **Run Immediately** and turn off **Notify When Run**. Tap **Next**.
6. Tap **New Blank Automation**.
7. Add the action **URL Encode**. Tap its input, choose **Shortcut Input**, then tap the variable again and pick **Content** (the SMS text).
8. Add the action **Open URLs**. Type `https://waliiidals.github.io/Everything-Budgeted/?sms=` and right after `=` insert the **URL Encoded Text** variable (from the suggestion bar).
9. Tap **Done**.

You can also copy the link from **Settings → Log from bank SMS** in the app.

### One-time check

1. In the app, open **Settings → Log from bank SMS** and paste one real purchase SMS into **Test with a real SMS**. It shows what would be logged (amount · shop · tag · date). If something is wrong, send the SMS text (you can mask the card digits) and I'll tune the reader.
2. Shortcuts opens links in **Safari**. The home-screen app and Safari keep separate storage on iPhone, so **sign in to your account once in Safari** too. Spends logged there sync to the home-screen app, which re-reads your account each time you switch back to it.

### What happens on each SMS

- Purchase → logged, with a toast and **undo** (e.g. "logged 3.000 KD · Ananas · Coffee").
- The same SMS twice → "already logged that one".
- Same day, amount and shop already logged by hand → logged, and the toast says so, so you can undo.
- OTP, refund, salary, transfer, ATM withdrawal or declined payment → skipped.
- A foreign-currency spend you haven't added under Settings → Other currencies, or an SMS it can't read → the Log a spend form opens, filled in as far as possible.
- Tag: the tag you always use for that shop, then a guess from the shop name, then "Other".
