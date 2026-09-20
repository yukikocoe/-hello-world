# Aruba ClearPass — Staff Portal Web Login page

- `staff-portal-login.html` — the page-content fragment to paste into
  **Guest > Configuration > Pages > Web Logins > [page] > Advanced > Edit the HTML**.
  Deployment steps and warnings are documented in the comment block at the
  top of the file — read it before pasting.
- `preview.html` — a standalone wrapper (adds `<!DOCTYPE>`/`<html>`/`<body>`)
  for previewing the design in a plain browser. Do not upload this file to
  ClearPass; use `staff-portal-login.html` for that.

## Quick deployment checklist

1. Guest > Configuration > Pages > Web Logins → create/edit the page, set
   **Vendor Settings** to your controller/gateway and the **Login Method**
   to your AD/RADIUS source, save once with the default generated form.
2. Reopen the page, use "Insert Login Form" / View Source to get the real
   generated `<form>` (with hidden MAC/SSID/switch-IP/redirect/CSRF fields).
3. In `staff-portal-login.html`, replace the placeholder `<form>...</form>`
   block with that generated form, keeping all hidden inputs untouched and
   only adding `cp-field` / `cp-field-input` / `cp-submit` classes to the
   visible username/password/submit elements.
4. Paste the whole fragment into the page's raw HTML content (not the Skin).
