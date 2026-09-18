# Completeness Reminder — e2e evidence

For [TheFeedFactory/ff-api#428](https://github.com/TheFeedFactory/ff-api/pull/428).

Run on 2026-09-18 against the ff-dev crabbox: ff-api on `feat/completeness-reminder`, ff-gui on
`feat/completeness-reminder-settings`, seeded from the scrubbed staging subset. Amsterdam&partners
(525 Locations) was switched on; every other Account was left off and stayed off.

Outgoing mail was captured by pointing `mailgun.apiUrl` at a local stub, so `MailgunService` posted
the same multipart form it would have posted to Mailgun. The two `.html` messages are byte-for-byte
what Mailgun would have received, headers included.

| | |
| --- | --- |
| index narrowing | 5883 Locations → 313 candidates |
| recipients | 2 — one NL with 6 Locations, one EN with 1 |
| ACL addresses with no User behind them | 202 |
| index drift caught by the database re-read | 3 |
| Locations with gaps but nobody to tell | 91 |
| ledger rows written | 7, one per recipient-and-Location pair, each with its message id |

## The message

`1-email-dev.png` — Dutch, six record cards. Only the empty rows are marked; the filled ones are
shown as the record, which is what makes it something to review rather than a list of complaints.
Two offers are flagged as unfinished ("Er staat nog geen kortingsbedrag bij", "De begin- of
einddatum ontbreekt") and one is celebrated as a banner. Never both for the same offer.

`1-email-solo.png` — English, a single Location, almost entirely empty. The opening explains *why*
it is empty before asking for anything, and offers "is this not your location?" as an exit.

## Unsubscribe

`3-unsubscribe-1-confirm.png` — the GET renders and changes nothing, which is the defence against
mail scanners and prefetchers that follow links in email with nobody clicking.

`3-unsubscribe-2-done.png` — only the POST acts.

`3-unsubscribe-4-tampered.png` — a tampered token returns a byte-identical body, so the page is no
oracle for who exists.

## ff-gui

`4-account-settings.png` — the per-Account switch with all six checks. The sixth, Aanbieding, is
visible because this Account has promotion products; an Account without them does not get it.

`4-user-preferences.png` — both email preferences, each labelled with what it governs. The
Completeness Reminder one is **off**, because the unsubscribe two screenshots earlier actually
landed. That is the end-to-end proof.

## The two follow-up lists

`5-unknown-addresses.png` and `5-unresponsive.png`, with their `X-Total-Count` header and the
"as far as we know" preamble that every download of the second list carries.

## What this run did NOT prove

- **Rendering with images blocked.** The screenshots came out byte-identical to the normal ones,
  because this Account has no logo configured, so the message contained no remote image to block.
  The check needs an Account with branding.
- **`Reply-To`.** No header went out, which is correct — Amsterdam&partners has no
  `brandingSettings.contactInfo.email` in the seed. Configure one before the first real send, or
  replies reach nobody.
- **Real delivery.** Nothing was sent to a real address; the mail service was a stub throughout.

The data is the scrubbed staging seed: every address is a fabricated `@ff-box.local` one, and the
venues are public Amsterdam institutions.
