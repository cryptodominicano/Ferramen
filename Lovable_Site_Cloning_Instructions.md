# Lovable Site Cloning Instructions

## Complete Guide to Exporting and Customizing a Lovable.dev Website

---

## Overview

This document outlines the step-by-step process for cloning a website built on Lovable.dev, removing Lovable branding, and customizing it for independent hosting.

---

## Step 1: Download the Built Website Files

### 1.1 Get the HTML File
Export or copy the `index.html` from your Lovable project.

### 1.2 Identify Required Assets
Open `index.html` and look for JavaScript and CSS references in the `<head>` section:

```html
<script type="module" src="/assets/index-XXXXX.js"></script>
<link rel="stylesheet" href="/assets/index-XXXXX.css">
```

### 1.3 Download Assets Using curl
Create an `assets` folder and download the files:

```bash
mkdir assets
cd assets
curl -o index-XXXXX.js "https://your-project.lovable.app/assets/index-XXXXX.js"
curl -o index-XXXXX.css "https://your-project.lovable.app/assets/index-XXXXX.css"
```

### 1.4 Find and Download Additional Assets
Search the JavaScript file for image references:

```bash
grep -oE '/assets/[a-zA-Z0-9_-]+\.[a-zA-Z0-9]+' assets/index-XXXXX.js
```

Download any found images (e.g., hero backgrounds):

```bash
curl -o hero-bg-XXXXX.jpg "https://your-project.lovable.app/assets/hero-bg-XXXXX.jpg"
```

---

## Step 2: Update HTML File Paths

### 2.1 Change Absolute Paths to Relative
Update the `index.html` to use relative paths:

**Before:**
```html
<script type="module" src="/assets/index-XXXXX.js"></script>
<link rel="stylesheet" href="/assets/index-XXXXX.css">
<link rel="icon" href="/favicon.ico">
```

**After:**
```html
<script type="module" src="assets/index-XXXXX.js"></script>
<link rel="stylesheet" href="assets/index-XXXXX.css">
<link rel="icon" href="images/favicon.ico">
```

---

## Step 3: Remove Lovable Branding

### 3.1 Remove Lovable Badge Styles
Delete the entire `<style>` block containing `#lovable-badge`:

```html
<!-- DELETE THIS -->
<style>
    #lovable-badge {
        position: fixed;
        bottom: 10px;
        right: 10px;
        /* ... more styles ... */
    }
</style>
```

### 3.2 Remove Lovable Analytics Script
Delete this line:

```html
<!-- DELETE THIS -->
<script defer src="https://your-project.lovable.app/~flock.js"
        data-proxy-url="https://your-project.lovable.app/~api/analytics"></script>
```

### 3.3 Remove Lovable Badge from Body
Delete the entire badge element (large SVG block):

```html
<!-- DELETE THIS ENTIRE BLOCK -->
<a id="lovable-badge" target="_blank" href="https://lovable.dev/projects/...">
    <span style="color: #A1A1AA;">Edit with</span>
    <svg>...</svg>
    <button id="lovable-badge-close">...</button>
</a>
```

### 3.4 Remove Lovable Badge JavaScript
Delete this script block:

```html
<!-- DELETE THIS -->
<script>
    if (window.self !== window.top || navigator.userAgent.includes('puppeteer')) {
        var badge = document.getElementById('lovable-badge');
        // ... badge visibility logic ...
    }
</script>
```

### 3.5 Remove Lovable Twitter Image Meta Tag
Delete if present:

```html
<!-- DELETE THIS -->
<meta name="twitter:image" content="https://pub-xxx.r2.dev/...lovable.app....png" />
```

### 3.6 Verify Removal
Search for remaining Lovable references:

```bash
grep -ri "lovable" --include="*.html" --include="*.js" --include="*.css" .
```

---

## Step 4: Replace Logo with Custom Logo

### 4.1 Understanding React Logo Structure
Lovable sites use React. The logo is often rendered as styled text, not an image:

```html
<a href="/">
  <div class="flex items-center gap-2">
    <div class="w-10 h-10 bg-primary rounded-lg">
      <span class="font-bold text-xl">F</span>
    </div>
    <div class="flex flex-col">
      <span class="font-bold">FERRAMEN</span>
      <span class="text-xs">S.R.L.</span>
    </div>
  </div>
</a>
```

### 4.2 Add CSS Override for Custom Logo
Add this to the `<head>` section:

```html
<style>
  /* Hide the default text logo elements */
  header a[href="/"] > div > div.w-10,
  header a[href="/"] > div > div.flex.flex-col {
    display: none !important;
  }

  /* Add custom logo image */
  header a[href="/"] > div {
    background-image: url('assets/logo.png') !important;
    background-size: contain !important;
    background-repeat: no-repeat !important;
    background-position: center !important;
    min-width: 550px !important;
    min-height: 153px !important;
  }
</style>
```

### 4.3 Adjust Logo Size
Modify `min-width` and `min-height` values as needed:
- Small: 180px x 50px
- Medium: 360px x 100px
- Large: 550px x 153px

---

## Step 5: Integrate Contact Form with Formspree

### 5.1 Create Formspree Account
1. Go to https://formspree.io
2. Create a new form
3. Copy your form ID (e.g., `mqekbekq`)

### 5.2 Add Formspree Integration Script
Add this before `</body>`:

```html
<script>
  document.addEventListener('DOMContentLoaded', function() {
    const checkForm = setInterval(function() {
      const form = document.querySelector('form');
      if (form && !form.hasAttribute('data-formspree-modified')) {
        form.setAttribute('data-formspree-modified', 'true');

        const submitBtn = form.querySelector('button[type="submit"]');
        if (submitBtn) {
          submitBtn.addEventListener('click', function(e) {
            e.preventDefault();
            e.stopPropagation();
            e.stopImmediatePropagation();

            const formData = new FormData(form);
            const originalText = submitBtn.textContent;

            submitBtn.disabled = true;
            submitBtn.textContent = 'Enviando...';

            fetch('https://formspree.io/f/YOUR_FORM_ID', {
              method: 'POST',
              body: JSON.stringify(Object.fromEntries(formData)),
              headers: {
                'Accept': 'application/json',
                'Content-Type': 'application/json'
              }
            })
            .then(response => {
              if (response.ok) {
                form.querySelectorAll('input, textarea').forEach(el => {
                  el.value = '';
                });
                alert('¡Gracias! Su mensaje ha sido enviado.');
              } else {
                alert('Error al enviar. Por favor intente de nuevo.');
              }
            })
            .catch(error => {
              alert('Error al enviar. Por favor intente de nuevo.');
            })
            .finally(() => {
              submitBtn.disabled = false;
              submitBtn.textContent = originalText;
            });

            return false;
          }, true);
        }

        clearInterval(checkForm);
      }
    }, 500);

    setTimeout(() => clearInterval(checkForm), 10000);
  });
</script>
```

Replace `YOUR_FORM_ID` with your actual Formspree form ID.

---

## Step 6: Configure Open Graph Meta Tags

### 6.1 Create OG Image
- Recommended size: 1200x630 pixels
- Include your logo and brand name
- Save as `assets/og-image.png`

### 6.2 Add Complete OG Tags
Replace/add these meta tags in `<head>`:

```html
<!-- Open Graph / Facebook -->
<meta property="og:type" content="website" />
<meta property="og:url" content="https://yourdomain.com/" />
<meta property="og:title" content="Your Company | Your Tagline" />
<meta property="og:description" content="Your company description for social sharing." />
<meta property="og:image" content="https://yourdomain.com/assets/og-image.png" />
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />
<meta property="og:image:alt" content="Your Company Name" />
<meta property="og:locale" content="es_DO" />
<meta property="og:site_name" content="Your Company" />

<!-- Twitter -->
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:url" content="https://yourdomain.com/" />
<meta name="twitter:title" content="Your Company | Your Tagline" />
<meta name="twitter:description" content="Your company description." />
<meta name="twitter:image" content="https://yourdomain.com/assets/og-image.png" />
<meta name="twitter:image:alt" content="Your Company Name" />
```

---

## Step 7: Test Locally

### 7.1 Start Local Server
```bash
cd your-project-folder
npx serve .
```

### 7.2 Open in Browser
Navigate to `http://localhost:3000`

### 7.3 Test Checklist
- [ ] Website loads without errors
- [ ] Logo displays correctly
- [ ] No Lovable branding visible
- [ ] Contact form submits successfully
- [ ] All images load
- [ ] Navigation works

---

## Step 8: Domain and Hosting

### 8.1 Dominican Republic Domains (.do)
Purchase from NIC.do (https://nic.do):
- `.com.do` - Best for commercial businesses (DOP 800/year)
- `.do` - Premium option (DOP 2,000/year)
- `.org.do` - Non-profits
- `.net.do` - Network services

### 8.2 Recommended Hosting Options
- Netlify (free tier available)
- Vercel (free tier available)
- GitHub Pages (free)
- Traditional hosting (cPanel)

### 8.3 Update Domain References
Update all domain references in `index.html`:
- Canonical URL
- Open Graph URLs
- Structured Data URLs
- Any hardcoded links

---

## Final File Structure

```
your-project/
├── index.html
├── logo.png (root copy for SEO)
├── assets/
│   ├── index-XXXXX.js
│   ├── index-XXXXX.css
│   ├── hero-bg-XXXXX.jpg
│   ├── logo.png
│   └── og-image.png
├── images/
│   └── favicon.ico
└── css/ (optional, if separate)
    └── additional-styles.css
```

---

## Troubleshooting

### Form Not Working
- Check browser console for CORS errors
- Verify Formspree form ID is correct
- Ensure JSON content type is set

### Logo Not Displaying
- Verify file path is correct
- Check file extension matches
- Inspect element to see if CSS is being applied

### White Screen After Form Submit
- Use `stopImmediatePropagation()` to prevent React handling
- Don't use `form.reset()` - manually clear fields instead

### Images Not Loading
- Check network tab for 404 errors
- Verify all asset files were downloaded
- Ensure paths are relative, not absolute

---

## Testing Tools

- **OG Tags**: https://developers.facebook.com/tools/debug/
- **Twitter Cards**: https://cards-dev.twitter.com/validator
- **LinkedIn**: https://www.linkedin.com/post-inspector/
- **General SEO**: https://search.google.com/test/rich-results

---

## Document Info

- **Created**: December 27, 2025
- **Project**: FERRAMEN S.R.L. Website
- **Original Platform**: Lovable.dev
- **Target Domain**: ferramen.com.do

