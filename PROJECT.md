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
- **`www.ecosipboard.com` has no valid certificate, and neither waiting nor recycling the domain fixes it.** Proven 2026-08-18 14:47 UTC: the live cert still has serial `0632B43D…8889` and `notBefore Aug 17 22:23:48 GMT`, **identical to the original**, so the unset/re-set at 05:12 UTC produced no new certificate at all. GitHub reused the existing valid one, almost certainly because the unset and re-set were ~11 seconds apart with an identical domain value. It is valid until Nov 15, so GitHub has no reason to replace it before renewal around mid-October. The root cause remains ordering: the original request went out at 2026-08-17 23:21 UTC while the domain had **no DNS records at all**, so there was no `www` to validate. `csinfraestructura.com` was claimed ~30 minutes later with DNS already correct and got `['csinfraestructura.com','www.csinfraestructura.com']`. **Ruled out as causes:** DNS (both zones are structurally identical, `www` CNAMEs to `nils-digital-com.github.io` on both, no CAA, no AAAA on either), and repo config (same branch, path and enforcement on both). **Do not retry the quick recycle, it is proven to be a no-op.** Real options: (a) delete the `www` CNAME in Hostinger so the name stops resolving instead of erroring, the zero-risk fix; (b) unset the custom domain, wait 10-15 minutes so the association actually lapses, then re-add, which costs that long as real downtime; (c) set the custom domain to `www.ecosipboard.com`, which is a different value so it forces a fresh request covering both names, but flips the canonical to `www` and would require updating `canonical` and `og:url` in `index.html`; (d) wait for the mid-October renewal, which should pick up `www` now that DNS is stable.
- **Social preview images do not exist.** `og:image` and `twitter:image` were dropped rather than shipped as 404s, so link previews on WhatsApp, LinkedIn and Facebook render with no image. This matters because the funnel hands leads off over WhatsApp. Needs a real 1200x630 and 1200x675 from Ignacio, or generated from brand assets, then uploaded to the GHL media library or committed here.
- **The old GHL funnel is still published.** Left in place deliberately as rollback. Archive it once the domain has been stable for about two weeks, and only then delete the pasted head-code blob discussion from `018`'s open threads.
- **Google Search Console.** Canonical moved from `https://ecosipboard.com/inicio` to `https://ecosipboard.com/`. Worth a re-submit once DNS settles.
- **Localised pages were advertised and never built.** The dropped `hreflang` tags implied Peru, Panama and English versions. If those are still wanted they are a real build, not a tag.
