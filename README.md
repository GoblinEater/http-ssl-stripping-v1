# HTTP Request Flow Visualizer

An interactive, single-file visualizer for HTTP, HTTPS, and the SSL stripping attack, built for **Week 10a** of the Goblin Eaters Cyber Cavern series "How Protocols Work, and How They Break."

It walks a request from browser to server two ways: once over HTTPS as it should go, and once through an on-path attacker who keeps the victim on plain HTTP. The point it makes is that SSL stripping defeats no cryptography. It exploits the fact that the upgrade to HTTPS is optional and the first request is plaintext.

## What it demonstrates

HTTP was built for a trusted reader, stateless and in the clear. HTTPS was added later as a privacy layer around the same protocol. SSL stripping lives in the gap between the two: an attacker already on the path answers the victim's first plaintext request, keeps the victim on HTTP, and relays to the real server over HTTPS. No cipher is broken and no certificate is forged. The data is genuine and simply travels in the clear.

The tool makes the two-channel nature visible: the victim-side hop stays plaintext while the attacker-to-server hop is fully encrypted.

## How to use it

Open `http-ssl-stripping-v1.html` in any modern browser. No build step, no dependencies beyond two web fonts loaded from Google Fonts.

- **Next / Prev** walk the steps.
- **Reset** returns to the start.
- **Auto Play** advances automatically.
- **HTTPS (Protected) / SSL Stripping** toggle (top right) switches between the two flows. Each flow has its own step sequence and its own dissector messages.

The dissector (lower left) shows the HTTP or TLS message on the wire at each step. On the stripped login step it shows an "On the Wire (HTTP)" column with the captured credentials beside an "Under HTTPS" column showing what the same fields would be inside TLS.

## The two flows

**HTTPS (Protected)**

1. Initial request over plain HTTP, before the browser knows the site requires HTTPS.
2. Server responds with a 301 redirect to HTTPS and an HSTS header.
3. TLS handshake, after which the channel is encrypted.
4. Login POST travels inside TLS as opaque ciphertext.

**SSL Stripping**

1. Prerequisite: the attacker is already on the path, by the Layer 2 means covered in the Week 2 post or a rogue access point.
2. The initial plain HTTP request lands on the attacker, not the server.
3. The attacker opens a real, valid HTTPS connection to the server.
4. The attacker rewrites every https link to http and serves the page to the victim over plain HTTP.
5. The victim submits the login form over HTTP, and the attacker captures the credentials in cleartext.
6. Closing step: why HSTS, HTTPS-first browsing, and browser warnings have narrowed the attack today.

## Design notes

- Built on the step-engine and dissector architecture shared with the ARP, TCP flag, and Finger tools in this series, so it matches their look and interaction model.
- Link colors carry the lesson: a green link with a closed-lock glyph marks an encrypted hop, a red open-lock marks a cleartext hop. In the stripping flow the victim side stays red while the server side is green.
- Typefaces: JetBrains Mono and Syne. Dark palette. Displayed text uses simple hyphens, not em-dashes. (Em-dashes remain only in the browser-tab title and in CSS comments, neither of which renders in the tool body.)
- All CSS `font-size` values use scalable units (`rem`), with the base set on the `html` element. Canvas text uses pixel-based font strings because the canvas rendering API requires them.
- The dissector badge is EXPOSED, not FALSIFIED. SSL stripping downgrades the channel rather than falsifying any field, so the data shown is genuine and simply in the clear. This keeps the framing accurate and distinguishes the attack from spoofing-style attacks elsewhere in the series.

## Data disclaimer

The host name, the sample login (`G. Eater` / `correct-horse-9`), and all message contents are fictional and were invented to illustrate the flow. They do not represent real people, credentials, or systems.

## Accuracy and verification notes

A few claims are worth confirming against a primary source before relying on them in print:

- **SSL stripping attribution.** The attack is Moxie Marlinspike's sslstrip, commonly dated to 2009. Verify the attribution and year if you cite them.
- **HSTS behavior.** The claim that preloading prevents the initial cleartext request is accurate to my knowledge. HSTS is specified in RFC 6797 (2012). Confirm the RFC number if you cite it.
- **SNI visibility.** The TLS step states the server name is visible via SNI. True by default, but Encrypted Client Hello (ECH) can conceal it, and ECH deployment may have shifted since this was written. Check current status if the point matters.
- **TLS handshake detail.** The handshake is shown as ClientHello, ServerHello, Certificate, Finished, at a conceptual level. Exact message flow differs between TLS 1.2 and 1.3. Verify if the prose goes into specifics.

## Deployment

Per the GECC publishing workflow for standalone tools:

1. Keep this repo under `~/code/goblin-eater/http-ssl-stripping/` with its own GitHub remote. Check `git remote -v` before pushing.
2. Copy the HTML file to the Hugo static tools directory: `~/ubritsa/static/tools/`.
3. Run the `publish` step to deploy via Cloudflare Pages.

If the working file state ever diverges from what the viewer shows, create a fresh-named copy (`http-ssl-stripping-v2.html`, and so on) rather than re-presenting the same filename.

## Status

Built from the series plan ahead of the Week 10a prose, then checked against the locked prose for consistency. Both center on SSL stripping and agree. If the post is later revised to emphasize a different attack, the tool may need to be repointed.

---

Goblin Eaters Cyber Cavern | ubritsa.com
