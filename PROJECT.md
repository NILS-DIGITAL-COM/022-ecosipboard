# Ecosipboard: fire-resistant industrial panel landing site (Spanish, Chile)

## What this is
The product landing site for **ECOSIPBOARD®**, a mineral construction panel sold by SPC Soluciones SPA, the same client as CS Infraestructura (Ignacio "Nacho" Silva). Spanish, `lang="es"`, B2B, aimed at site managers and engineers who need F30-F180 fire certification without rework. Lives on its own domain, `ecosipboard.com`.

Split out of `018-cs-infraestructura` on 2026-08-17 when the site moved off GoHighLevel page hosting. That repo keeps `csinfraestructura.com`; this one owns `ecosipboard.com`. **The split was forced by GitHub Pages: one custom domain per repo.**

## Architecture
- `index.html` : the landing page. Single self-contained file, embedded CSS and JS, green palette, Inter + Space Grotesk.
- `gracias-ecosip-page/index.html` : thank-you page, the **live redirect target** of the GHL survey.
- `gracias-ecosip/index.html` : byte-identical copy of the thank-you page. Both paths resolved on the old GHL funnel, so both are served here to avoid breaking any existing link.
- `inicio/index.html` : redirect to `/`. Old funnel step path, kept so existing links and any indexed URL still land somewhere real.
- `CNAME` : `ecosipboard.com`
- `.nojekyll` : skip Jekyll processing
- Hosting: **GitHub Pages**, `main` branch, root. No build step.
- Lead capture: GHL **survey** widget `qYXpFPOvodCF6Znh0QJL`, embedded from `link.nilsdigital.com`. Leads still flow into the GHL CRM, pipeline "Ecosipboard" (stages eco1/eco2/eco3/Ganado/Perdido). Only page hosting left GHL, the CRM did not.
- Tracking: Google Ads `AW-17979536691` (gtag, direct in the page), GTM container `GTM-K8VJRZQL`, Microsoft Clarity `wkeh5uuh03`, and the `portal.nilsdigital.com` analytics script.

## Conventions
- Spanish, formal B2B tone. Client-facing copy only ever contains what the client actually supplied.
- Green palette: `#1a3d2b`, `#2d5a3d`, `#4a9c6d`, mint `#7ac9a0`. Fonts Inter + Space Grotesk.
- The hero eyebrow rotates between audience variants ("jefes de obra", "ingenieros"), so screenshots of the same build will legitimately differ.
- **Never put a viewport meta tag into a GoHighLevel head-code field.** See the changelog below for why. Not relevant to this repo any more, but it is the reason this repo exists.

## Changelog
## 2026-08-17 Split out of 018 and migrated from GoHighLevel to GitHub Pages

Ecosipboard was served by a GHL funnel that iframed `nils-digital-com.github.io/018-cs-infraestructura/ecosipboard.html` at `width: 100vw`. The wrapper emitted `<meta name="viewport" content="width">`, an invalid value, so phones fell back to the legacy 980px desktop viewport, the iframe became 980 CSS px wide, and the page's mobile breakpoints never fired. **The site rendered its full desktop layout on every phone.** Root cause was a whole `<!DOCTYPE html><html><head>...` document pasted into the funnel page's Head Tracking Code: GHL hoists the viewport meta out of it into its own head slot and truncates the value at the first `=`. Serving the page directly removes the wrapper, the iframe and the bug in one move.

Built this repo from the existing files, no content retyped:
- `index.html` is `018-cs-infraestructura/ecosipboard.html` with a `<head>` injection only.
- **Ported the SEO block that previously existed only inside GHL.** `ecosipboard.html` had no description, canonical, Open Graph or JSON-LD of its own, all of it lived in the funnel's head code and would have been silently lost. Extracted the live field out of the served page's Nuxt payload, JSON-unescaped it, and merged it in. `GTM-K8VJRZQL` was also injected only by GHL and is now in the page, so the same set of tags fires as before.
- **Dropped every dead reference rather than carrying it over.** `og:image`, `twitter:image`, all three favicons, `/css/main.css` and the schema `logo` were all 404s on the old domain and would have stayed 404s here. The four `hreflang` alternates pointed at `/pe/inicio`, `/pa/inicio` and `/en/home`, which never existed and only ever served the home-page fallback.
- **Mirrored the old URL paths on purpose.** The survey's redirect is an absolute URL to `https://ecosipboard.com/gracias-ecosip-page`, so serving that exact path means **no GHL edit was needed** and no existing link broke.

Verified before DNS by pinning the hostname to the Pages IP (`--host-resolver-rules`), so the real served bytes were tested rather than a local copy: all four paths return 200 at the exact built sizes, mobile at 390x844 gives layout viewport 390px with no horizontal overflow and the survey iframe at 350px, desktop at 1440px has no overflow, `dataLayer` present, encoding clean with no mojibake.

**Outage note, worth remembering.** The DNS change was handed to a browser agent, which executed the delete half of the instructions and not the add half. It removed the apex `A` and the `www` `CNAME` for `ecosipboard.com` and stopped, leaving the domain with no address records at all. MX (Titan) and the SPF TXT survived, so email was never at risk. Lesson: **give a browser agent the additive step first, or hand it a single combined instruction it cannot half-complete.** Deleting before the destination exists has no rollback except retyping the old values, which are: apex `A` `162.159.140.166`, `www` `CNAME` `sites.ludicrous.cloud`.

## Open threads
- **`www.ecosipboard.com` certificate: re-issue requested 2026-08-18 05:12 UTC, awaiting provisioning.** The original cert was apex-only because it was requested at 2026-08-17 23:21 UTC, when the domain was claimed via `CNAME` but **had no DNS records at all** (the browser-agent incident had deleted them), so there was no `www` for GitHub to validate. `csinfraestructura.com` got both names because its records existed when its domain was claimed. Cause is ordering, not configuration: both repos are set identically. **Fix applied:** unset and re-set the custom domain at 05:12 UTC, the first request ever made with correct DNS in place. Re-`PUT`ting the same `cname` value beforehand did nothing, GitHub treats it as a no-op, so the full unset/re-set is the step that matters. Verify with `gh api repos/NILS-DIGITAL-COM/022-ecosipboard/pages` and read `https_certificate.domains`; it should list both names. Until then `https://www.ecosipboard.com` gives a hostname mismatch while the apex is fine. If it has still not picked up `www` after a day, the fallback is to delete the `www` CNAME in Hostinger so the name does not resolve at all, which beats resolving to a certificate warning.
- **Social preview images do not exist.** `og:image` and `twitter:image` were dropped rather than shipped as 404s, so link previews on WhatsApp, LinkedIn and Facebook render with no image. This matters because the funnel hands leads off over WhatsApp. Needs a real 1200x630 and 1200x675 from Ignacio, or generated from brand assets, then uploaded to the GHL media library or committed here.
- **The old GHL funnel is still published.** Left in place deliberately as rollback. Archive it once the domain has been stable for about two weeks, and only then delete the pasted head-code blob discussion from `018`'s open threads.
- **Google Search Console.** Canonical moved from `https://ecosipboard.com/inicio` to `https://ecosipboard.com/`. Worth a re-submit once DNS settles.
- **Localised pages were advertised and never built.** The dropped `hreflang` tags implied Peru, Panama and English versions. If those are still wanted they are a real build, not a tag.
