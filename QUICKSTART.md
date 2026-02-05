# ⚡ Quick Start Guide - Cloud Image Notepad

## 5-Minute Setup

### Step 1: Create GitHub Token (2 minutes)

1. Go to [GitHub Settings → Developer Settings → Tokens](https://github.com/settings/tokens)
2. Click **"Generate new token (classic)"**
3. Set **Name**: "Cloud Image Notepad"
4. Set **Expiration**: 90 days or more
5. Check **`repo`** scope only
6. Click **"Generate token"**
7. **Copy the token** (appears once only!)

### Step 2: Create Repository (1 minute)

Option A: Use existing repository OR

Option B: Create new repository:
1. Go to [github.com/new](https://github.com/new)
2. **Repository name**: `cloud-image-notepad` (or any name)
3. **Visibility**: Public (images visible) or Private (token required)
4. Click **"Create repository"**

### Step 3: Open the App (2 minutes)

1. Open `index.html` in your web browser
2. Fill in configuration:
   - **Owner**: Your GitHub username
   - **Repository**: Repository name from Step 2
   - **Folder**: `images` (will be created automatically)
   - **Token**: Paste token from Step 1
3. Click **"🔄 Refresh List"** to test
4. Done! ✅

## Using the App

### Upload Images
```
Click "Select Image" → Choose file → Click "➕ Add Image"
```

### View Images
```
Click "🔄 Refresh List" → Gallery updates
```

### Delete Images
```
Click "🗑️ Delete" on any image → Confirm
```

### Update Images
```
Select file with same name → Click "➕ Add Image" → Automatically replaces old version
```

## Token Explanation in Simple Terms

**Token = Secure Password Alternative**

Instead of giving the app your actual GitHub password:
1. You create a special **temporary password** (token)
2. You give the app this token
3. GitHub uses the token to verify requests
4. If the token is stolen, you delete it and create a new one
5. Your real password stays safe

**Why?** Because if the token leaks, only images are at risk, not your entire GitHub account.

### Token Lifespan

```
Day 0: Create token → Copy to app
Days 1-90: Token works perfectly
Day 91+: Token expires → Need new token
```

When token expires:
1. App shows "❌ Unauthorized" error
2. Go to GitHub Settings → Delete old token
3. Create new token
4. Paste new token into app
5. Done! 

## Security Checklist

- [ ] Token has 90+ day expiration
- [ ] Token has only `repo` scope (nothing more)
- [ ] Token is NOT shared publicly
- [ ] Token is NOT in version control
- [ ] Repository folder exists or token can create it

## Common Issues & Fixes

| Issue | Fix |
|-------|-----|
| "Repository not found" | Check username and repo name spelling |
| "Unauthorized" | Token expired? Create new one |
| "Network error" | Check internet connection |
| "Upload failed" | Check file is valid image (JPG, PNG, GIF, etc) |

## Examples

### Example 1: Personal Photo Gallery
1. Create repo: `my-photos`
2. Token with `repo` scope
3. Upload JPG files
4. Share link with family

### Example 2: Design Assets
1. Create repo: `design-assets`
2. Keep token private
3. Upload PNG/SVG files
4. Use images in your projects

### Example 3: Project Documentation
1. Create repo: `project-docs`
2. Upload screenshots
3. Reference in README.md
4. Version control images with code

## Need Help?

### Check Token Details
1. GitHub → Settings → Developer settings → Tokens
2. Click token name to see details
3. Verify expiration date
4. Verify scope includes `repo`

### Check Browser Console for Errors
1. Press **F12** in browser
2. Go to **Console** tab
3. Look for red error messages
4. Copy error text for debugging

### Reset Everything
If something breaks:
1. Close browser tab
2. Open `index.html` again
3. Clear configuration fields
4. Enter credentials again
5. Click "Refresh List"

## Next Steps

- ✅ Create GitHub account (if needed)
- ✅ Generate token with 90+ day expiration
- ✅ Create repository
- ✅ Open `index.html`
- ✅ Enter configuration
- ✅ Upload your first image!

---

**That's it! You're ready to use Cloud Image Notepad** 🎉
