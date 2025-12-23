# Image Optimization Guide

This guide explains how to optimize images for the Fatboy Financial Planner website to improve mobile performance.

## Quick Start

### Option 1: Automated Script (Recommended)

1. **Install Node.js** (if not already installed)
   - Download from https://nodejs.org/ (LTS version recommended)
   - Verify: `node --version`

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Run optimization**
   ```bash
   npm run optimize-images
   ```

4. **Review results**
   - Optimized images will be in `assets/images/optimized/`
   - HTML snippets will be in `IMAGE_SNIPPETS.html`

---

## What the Script Does

### 1. Logo Optimization
- Resizes logo to 320px width (2x for retina displays)
- Compresses PNG with quality: 90
- Generates WebP version
- **Expected savings**: ~60-70% reduction

### 2. Screenshot Optimization
Creates 3 sizes of each screenshot:
- **400w** - For mobile devices (< 600px)
- **800w** - For tablets (600px - 1024px)
- **1200w** - For desktop (> 1024px)

Each size generated in:
- **PNG** - Compressed with quality: 85
- **WebP** - Modern format, ~30% smaller than PNG

### 3. Performance Benefits

**Before optimization:**
- Total images: ~3.3MB
- Largest file: 515KB
- Mobile users download full-size images

**After optimization:**
- Mobile (400w): ~40-80KB per image
- Tablet (800w): ~100-150KB per image
- Desktop (1200w): ~150-200KB per image
- **Total savings**: 60-80% reduction in data transfer

---

## Understanding Responsive Images

### The `<picture>` Element

```html
<picture>
  <!-- WebP version (modern browsers) -->
  <source srcset="/assets/images/optimized/plan_summary-400w.webp 400w,
                  /assets/images/optimized/plan_summary-800w.webp 800w,
                  /assets/images/optimized/plan_summary-1200w.webp 1200w"
          sizes="(max-width: 600px) 100vw, (max-width: 1024px) 80vw, 1200px"
          type="image/webp">

  <!-- PNG fallback (older browsers) -->
  <source srcset="/assets/images/optimized/plan_summary-400w.png 400w,
                  /assets/images/optimized/plan_summary-800w.png 800w,
                  /assets/images/optimized/plan_summary-1200w.png 1200w"
          sizes="(max-width: 600px) 100vw, (max-width: 1024px) 80vw, 1200px"
          type="image/png">

  <!-- Fallback for very old browsers -->
  <img src="/assets/images/optimized/plan_summary-800w.png"
       alt="Plan Summary Dashboard"
       loading="lazy"
       style="max-width: 100%; height: auto;">
</picture>
```

### How Browsers Choose

1. **Format**: WebP if supported, otherwise PNG
2. **Size**: Based on viewport width and `sizes` attribute
   - Mobile (< 600px): Uses 400w image
   - Tablet (600-1024px): Uses 800w image
   - Desktop (> 1024px): Uses 1200w image

---

## Manual Optimization (Alternative)

If you prefer not to use the Node.js script:

### Using Online Tools

1. **Squoosh** (https://squoosh.app/)
   - Drag and drop images
   - Adjust quality slider to 85
   - Download WebP and optimized PNG
   - Manual but easy to use

2. **TinyPNG** (https://tinypng.com/)
   - Batch upload up to 20 images
   - Automatic optimization
   - No WebP generation (PNG only)

### Using ImageMagick (Command Line)

```bash
# Install ImageMagick
# Windows: choco install imagemagick
# Mac: brew install imagemagick
# Linux: apt-get install imagemagick

# Optimize single PNG
magick input.png -resize 800x -quality 85 output-800w.png

# Convert to WebP
magick input.png -resize 800x -quality 85 output-800w.webp

# Batch process all PNGs
for file in *.png; do
  magick "$file" -resize 400x -quality 85 "${file%.png}-400w.png"
  magick "$file" -resize 800x -quality 85 "${file%.png}-800w.png"
  magick "$file" -resize 1200x -quality 85 "${file%.png}-1200w.png"
  magick "$file" -resize 400x -quality 85 "${file%.png}-400w.webp"
  magick "$file" -resize 800x -quality 85 "${file%.png}-800w.webp"
  magick "$file" -resize 1200x -quality 85 "${file%.png}-1200w.webp"
done
```

---

## Updating Your Markdown Files

After optimization, update your image references:

### Before
```markdown
![Plan Summary Dashboard](/assets/images/plan_summary.png)
```

### After
```html
<picture>
  <source srcset="/assets/images/optimized/plan_summary-400w.webp 400w,
                  /assets/images/optimized/plan_summary-800w.webp 800w,
                  /assets/images/optimized/plan_summary-1200w.webp 1200w"
          sizes="(max-width: 600px) 100vw, (max-width: 1024px) 80vw, 1200px"
          type="image/webp">
  <source srcset="/assets/images/optimized/plan_summary-400w.png 400w,
                  /assets/images/optimized/plan_summary-800w.png 800w,
                  /assets/images/optimized/plan_summary-1200w.png 1200w"
          sizes="(max-width: 600px) 100vw, (max-width: 1024px) 80vw, 1200px"
          type="image/png">
  <img src="/assets/images/optimized/plan_summary-800w.png"
       alt="Plan Summary Dashboard"
       loading="lazy"
       style="max-width: 100%; height: auto;">
</picture>
```

**Tip**: The script generates all these snippets for you in `IMAGE_SNIPPETS.html`!

---

## Testing Optimizations

### 1. Visual Quality Check
- Open optimized images
- Compare with originals
- Ensure no visible degradation
- Screenshots should still be crisp and readable

### 2. File Size Check
```bash
# Original sizes
ls -lh assets/images/*.png

# Optimized sizes
ls -lh assets/images/optimized/*.webp
```

### 3. Browser Testing

**Chrome DevTools:**
1. Open DevTools (F12)
2. Go to Network tab
3. Filter by "Img"
4. Reload page
5. Check:
   - Mobile loads 400w versions
   - Desktop loads 1200w versions
   - WebP is used (if supported)

**Lighthouse Audit:**
1. Open DevTools (F12)
2. Go to Lighthouse tab
3. Select "Mobile" device
4. Run audit
5. Look for:
   - "Properly size images" ✅
   - "Serve images in next-gen formats" ✅
   - "Defer offscreen images" ✅

---

## Troubleshooting

### Script fails to run

**Error**: `Cannot find module 'sharp'`
- **Solution**: Run `npm install`

**Error**: `EACCES: permission denied`
- **Solution**: Run with admin privileges or fix folder permissions

### Images look blurry

**Problem**: Quality too low
- **Solution**: Increase quality in `optimize-images.js` (line 20-24)
- Change from 85 to 90 or 95

### Images too large still

**Problem**: Quality too high
- **Solution**: Decrease quality to 75-80
- Or use more aggressive WebP compression

### Jekyll not showing optimized images

**Problem**: Path incorrect
- **Solution**: Ensure paths start with `/assets/images/optimized/`
- Rebuild Jekyll: `bundle exec jekyll serve`

---

## Performance Metrics

### Expected Improvements

**Before Optimization:**
- First Contentful Paint: ~2.5s
- Largest Contentful Paint: ~4.2s
- Total Blocking Time: ~500ms
- Lighthouse Score: ~65-75

**After Optimization:**
- First Contentful Paint: ~1.2s ⚡ -52%
- Largest Contentful Paint: ~2.1s ⚡ -50%
- Total Blocking Time: ~200ms ⚡ -60%
- Lighthouse Score: ~85-95 ⚡ +20 points

---

## Maintenance

### Adding New Screenshots

1. Save new screenshot to `assets/images/`
2. Add filename to `SCREENSHOTS` array in `optimize-images.js`
3. Run `npm run optimize-images`
4. Copy HTML snippet from `IMAGE_SNIPPETS.html`
5. Update your markdown file

### Updating Existing Images

1. Replace file in `assets/images/`
2. Delete old optimized versions
3. Run `npm run optimize-images`
4. No code changes needed (paths stay the same)

---

## Cost-Benefit Analysis

### One-Time Setup
- Time: 15-30 minutes
- Technical difficulty: Low (just run npm commands)
- Cost: Free (open source tools)

### Long-Term Benefits
- **60-80% faster** image loading
- **Better SEO** (Core Web Vitals)
- **Lower bandwidth costs** for users
- **Improved user experience** on mobile
- **Higher conversion rates** (faster site = more engagement)

### Real-World Impact

For a user on 4G mobile (4 Mbps):
- **Before**: 6.6 seconds to load all images
- **After**: 1.3 seconds to load all images
- **Savings**: 5.3 seconds ⚡

---

## Additional Resources

- [WebP Documentation](https://developers.google.com/speed/webp)
- [Responsive Images Guide](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)
- [sharp Documentation](https://sharp.pixelplumbing.com/)
- [Lighthouse Performance](https://developer.chrome.com/docs/lighthouse/performance/)

---

**Questions?** Check `MOBILE_OPTIMIZATION_SUMMARY.md` for mobile-specific optimizations.
