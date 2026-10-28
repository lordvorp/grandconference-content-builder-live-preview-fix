# Bug report / patch: restore legacy Content Builder Live Preview (GrandConference 5.3.7)

**Theme:** GrandConference  
**Broken in:** 5.3.7  
**Last known working:** 5.2.2 (same preview markup/handlers still present there)  
**Reporter:** Rob Bloodgood  

## Summary

The theme’s **legacy visual editor** (built-in Content Builder — not Gutenberg, not Elementor) **Live View** shows a blank right-hand pane. Device switcher UI is still present, but the live preview iframe and its wiring were removed from the parent theme in 5.3.7.

We understand ThemeGoods is investing in Elementor. That direction is welcome; this package only returns the pieces that make the legacy visual editor Live Preview work again for sites that still use it. The Elementor push appears to have coincided with (or caused) stripping Live Preview DOM/JS while leaving half the stack behind (`page-preview-page.php`, device toolbar CSS, AJAX in `grandconference-custom-post`).

## Reproduction

1. Install GrandConference **5.3.7** with a Content Builder page that has modules.
2. Edit the page in WP admin → Content Builder.
3. Click **Live View**.
4. Observe: left module list works; right preview area is empty gray background.
5. In DevTools, search for `#ppb_live_preview_frame` — it is missing from the DOM.

## Root cause

Compared with 5.2.2, version **5.3.7 removed** required Live Preview pieces while leaving related CSS/JS and `page-preview-page.php` in place:

| File | What was removed |
|------|------------------|
| `modules/content_builder.php` | `#ppb_preview_page` control; `#ppb_live_preview_frame` iframe markup |
| `functions/theme_script.js` | `#ppb_preview_page` click handler; `#ppb_preview_devices` click handler; clear of `#ppb_preview_page.active` when leaving live mode |
| `page.php` | Route for `?ppb_preview_page=…` → `page-preview-page.php` |

AJAX handlers in **grandconference-custom-post** (`grandconference_ppb_preview_page_set_data`, etc.) are still present. This is a parent-theme regression, not a plugin gap.

## Fix

Apply `grandconference-5.3.7-restore-content-builder-live-preview.patch` from the theme root:

```bash
cd wp-content/themes/grandconference
patch -p1 < grandconference-5.3.7-restore-content-builder-live-preview.patch
```

The patch restores the 5.2.2 Live Preview markup and JS handlers (and the `page.php` preview route) without other theme changes.

## Suggested verification

1. Open a Content Builder page → Live View → iframe loads the page with `ppb_preview_page=true&ppb_live=true`.
2. Preview / device icons switch `#ppb_live_preview_frame_wrapper` `rel` and reload the frame.
3. Classic (non-live) Preview button still opens the Fancybox preview.

## Optional hardening (not in this patch)

On hosts with full-page cache (LiteSpeed, W3TC, etc.), preview URLs can be cached empty. Sending `nocache_headers()` / `DONOTCACHEPAGE` when `ppb_preview_page` or `ppb_preview` is present avoids that. Separate from the missing-iframe bug.

## Out of scope for this report

Site-specific child-theme customizations (Performer/Vendor labels, email social icon, vendor outbound links, Font Awesome 5 icon CSS) are **not** part of this fix.
