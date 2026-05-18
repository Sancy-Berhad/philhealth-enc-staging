# philhealth-enc-staging

Public host for AES-encrypted attachment `.enc` files served to **PhilHealth eClaims 3.0 staging** via the `pDocumentURL` field in claim XML.

This repo exists only because PhilHealth's staging fetcher needs publicly-reachable HTTPS URLs to pull attachment files from. GitHub Pages with `.nojekyll` gives us:

- Real binary serving (no Jekyll Markdown/HTML transforms)
- Proper `Content-Type` by extension (`.enc` → `application/octet-stream`)
- Persistent stable URLs
- Free, no PII concerns at our test scale

## Layout

```
.nojekyll                       ← tells Pages not to run Jekyll
verify.txt                      ← sentinel for curl-testing the site is live
attachments/round0/<file>.enc   ← per-round encrypted attachment payloads
attachments/round1/...
...
```

URL pattern that goes into `pDocumentURL`:

```
https://sancy-berhad.github.io/philhealth-enc-staging/attachments/<round>/<file>
```

## Caveats

- **TEST ARTEFACTS ONLY.** Every `.enc` here is encrypted with the staging `facilityCipherKey` (`PHilheaLthDuMmy270329`). DO NOT upload anything produced with a production cipher key or anything containing real patient PII.
- Build delay after a push to `main` is 30–60 seconds. PhilHealth may also cache; force-bust with unique filenames per round (`round0_*`, `round1_*`) rather than reusing names.
- Traffic stats (Insights → Traffic) refresh hourly and don't surface User-Agent or Range headers — only paths and counts. Migrate to Cloudflare Pages if richer fetch metadata is needed.

## Why this exists

See the parent test plan at
`/Users/edgardoguevarrajr/.claude/plans/clever-mapping-rocket.md`
and the audit memory at
`reference_philhealth_attachment_encryption.md`.

The encryption tooling that produces files here is
`/Users/edgardoguevarrajr/repo/IDEAS_PhilHealth_Attachment_PostmanEncrypt.mjs`,
a verbatim port of the working `uploadeClaims` pre-request script from
`PhilHealth eClaims (Staging).postman_collection.json`.
