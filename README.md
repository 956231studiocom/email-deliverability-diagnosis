# Email Deliverability Diagnosis: Open Rate Fell and You Cannot See Why

A short diagnostic for stores whose email stopped landing. Not a tool, not a service — a decision tree and the three things that actually cause it.

If your open rate dropped from 60% to 32% and you already tried changing the subject line, the send time, and cleaning the list, none of those were the cause. Those are fixes for a content problem. A placement problem needs a different check.

---

## Open rate is the wrong first metric

Open rate is now dominated by Apple Mail Privacy Protection and by image blocking. A mailbox provider that prefetches your images logs an "open" whether a human ever saw the message. Your 60% may never have been people.

Use open rate as a trend, not an absolute. If it collapses by half within a few sends, treat it as a placement signal and go straight to the checks below.

---

## The three real causes

Almost every "we stopped landing" case is one of these three. They look identical from the dashboard and have completely different fixes.

### 1. Authentication is missing or misaligned

Not "you have SPF". **Aligned.** A record can exist, pass, and still be misaligned with the From domain — which is what actually decides placement at Gmail and Microsoft.

The check that matters is in a real message header, not in a tool summary:

```
Authentication-Results: mx.google.com;
   spf=pass smtp.mailfrom=send.yourdomain.com;
   dkim=pass header.d=yourdomain.com;
   dmarc=pass header.from=yourdomain.com
```

What you are looking for:

- `spf=pass` where `smtp.mailfrom` is **the same organisational domain** as your From address (or a designated subdomain with proper alignment)
- `dkim=pass` where `header.d` equals your From domain
- `dmarc=pass` with `header.from` matching what recipients see

A common failure: you send through a platform, SPF passes for the platform's domain, DKIM signs for the platform's domain, and DMARC has no aligned identifier for **your** domain. Everything says "pass" and you still land in spam.

**Fix shape:** add the platform's DKIM key so it signs for your domain, or send through a subdomain with its own aligned records. Do not "just add an SPF include" — that fixes authentication but not alignment.

### 2. Sudden volume spike

Sending 10× your normal daily volume overnight reads as a compromised account or a new sender. Filters throttle first and ask never.

**Fix shape:** ramp. A 14-day schedule that increases daily volume while keeping engagement high enough to sustain the ratio — the practical rule is roughly two engaged sends for every cold one, maintained permanently, not for two weeks.

### 3. List hygiene

A list with no engagement signal gets treated as a list of strangers. Sending to the whole thing repeatedly converts your domain reputation into a spam signal.

**Fix shape:** prune on engagement, not on time. Subscribers who have not opened in 90–180 days go into a re-engagement sequence; those who still do not respond are removed. Selling or importing a list accelerates this to weeks.

---

## Diagnostic order

Run in this order. Each step changes what the next step means.

1. **Pull one raw message header** from a recent send. Not a screenshot, not a tool report. Look at the three `Authentication-Results` lines.
2. **Confirm DKIM is signing for your From domain.** If not, nothing else matters yet.
3. **Check whether the drop is one provider or all of them.** Gmail, Outlook and Apple Mail fail differently. A fix that works for Gmail routinely fails for Outlook because they weight engagement and complaint rates differently.
4. **Check send volume against your 30-day baseline.** Any 3× jump in a day is enough to explain a collapse on its own.
5. **Check bounce rate over the last 7 days.** Above 3% means stop sending and fix authentication first — continuing to send trains filters against you, and that damage outlasts the campaign.

Only after all five should you look at content, subject lines, or send times.

---

## What to check when a single provider fails

| Symptom | Likely cause | First thing to verify |
|---|---|---|
| Gmail spam, others fine | DMARC alignment or engagement | `header.d` matches From domain |
| Outlook spam, others fine | Reputation / IP or domain age | Sending history length on that domain |
| Apple Mail only | Not placement — prefetch inflating opens | Compare click rate, not open rate |
| Everything degraded at once | Volume spike or complaint rate | Daily volume vs 30-day baseline |
| Bounces above 3% | List quality or authentication | Full SPF lookup count (10-limit) and invalid addresses |

---

## Never do these

- **Never** buy or scrape a list. It converts a fixable reputation problem into an unrecoverable one.
- **Never** resend the same message to non-openers repeatedly. It reads as pressure and raises complaints.
- **Never** treat a tool's "deliverability score" as evidence. Scores are modelled; headers are recorded.
- **Never** promise inbox placement. Nobody can — the mailbox provider decides. What you control is alignment, volume, and engagement.

---

## Full kit

This is the diagnosis. The paid kit is the fix, in copy-paste form:

- Inbox placement diagnostic across Gmail, Outlook and Apple Mail, ranked by provider
- SPF / DKIM / DMARC setup walkthrough with the exact DNS records and how to read a DMARC aggregate report when a sending source fails alignment
- The 14-day warm-up schedule, including the warm-to-cold ratio to maintain permanently
- Spam-trigger word list — the phrases that quietly tank placement
- Re-engagement sequence template to win back non-openers before you prune them

<https://475563839418.gumroad.com/l/deliverability-rescue-kit>

Related, free: the [15-check store ops daily checklist](https://github.com/956231studiocom/store-ops-daily-checklist) — check #4 and #5 are the email ones, with the daily verification routine.

---

## Contributing

If you have a header pattern that explains a placement failure not covered here, open an issue with the redacted `Authentication-Results` block. Real cases get added.

## License

CC BY 4.0 — copy, translate, reuse with attribution.
