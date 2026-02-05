# ☁️ Cloud Image Notepad

A lightweight, browser-based image manager that stores images directly in GitHub, with no backend server required.

## 📋 Features

- **Cloud Storage**: Save images to a GitHub repository
- **Image Gallery**: View all images from your cloud folder
- **Upload & Replace**: Add new images or update existing ones
- **Delete Images**: Remove images from cloud storage
- **Responsive Design**: Works on desktop, tablet, and mobile devices
- **Local Credentials**: Saves configuration locally (owner, repo, folder)
- **Error Handling**: Graceful error messages for network issues, auth failures, and invalid configs
- **Base64 Encoding**: Automatically encodes images for GitHub API compatibility

## 🔐 How GitHub Tokens Work

### What is a Token?

A **GitHub Personal Access Token** is a secure credential that acts like a password. It allows the Cloud Image Notepad app to authenticate with GitHub on your behalf without storing your actual GitHub password.

**Security Principle**: Instead of giving the app your password (which is dangerous), you give it a limited-access token that:
- Can only access specific repositories
- Expires after a set time (90+ days)
- Can be revoked instantly if compromised
- Is much safer than sharing your actual password

### Token Authentication Flow

```
User App  →  GitHub API  →  GitHub Server
  ↓
[User provides token]
  ↓
PUT /repos/{owner}/{repo}/contents/{path}
Authorization: token ghp_xxxxxxxxxxx
  ↓
[GitHub validates token]
  ↓
✅ Authorized → Create/Update File
OR
❌ Denied → Return 401 Unauthorized
```

### Creating a GitHub Token

**Step-by-Step Guide:**

1. **Go to GitHub Settings**
   - Log in to GitHub → Click your profile icon (top right) → Settings

2. **Find Developer Settings**
   - Scroll down to find "Developer settings" (at the bottom left)
   - Click "Personal access tokens" → "Tokens (classic)"

3. **Generate New Token**
   - Click "Generate new token (classic)" button

4. **Configure Token Permissions**
   - **Token name**: "Cloud Image Notepad" (for your reference)
   - **Expiration**: Set to 90 days or more (as per requirements)
   - **Scopes**: Select only `repo` scope
     - `repo` = Full control of private repositories
     - This includes read/write access to repository contents

5. **Generate and Copy**
   - Click "Generate token" at the bottom
   - **Important**: Copy the token immediately! You won't see it again.
   - If you lose it, you must generate a new one.

### Token Expiration

Tokens expire after the set period (minimum 90 days recommended). When expired:
- The app will fail with "401 Unauthorized" error
- You must generate a new token in GitHub settings
- Paste the new token into the app

### Security Best Practices

1. **Never Share Your Token**
   - Don't commit it to version control
   - Don't share it in emails or messages
   - Don't post it on public forums

2. **If Token is Compromised**
   - Go to GitHub Settings → Personal access tokens
   - Click "Delete" next to the token
   - Generate a new token immediately

3. **Scope Limitation**
   - Only give the app `repo` scope (it needs this for image storage)
   - Don't enable additional scopes like `admin` or `delete_repo`

4. **Regular Rotation**
   - Consider rotating tokens every 6-12 months
   - Set a calendar reminder for token expiration dates

## 🚀 Getting Started

### Prerequisites

- A GitHub account (free)
- A GitHub repository to store images
- A Personal Access Token (see above)
- A modern web browser

### Setup Instructions

#### 1. Create a GitHub Repository

```bash
# Option A: Use existing repository
# OR

# Option B: Create a new repository
# Go to github.com/new and create a repository named "cloud-image-notepad"
```

#### 2. Create Images Folder

In your repository:
```bash
# Create a folder named "images" for storing your images
# (GitHub will create it automatically when you upload the first file)
```

#### 3. Create GitHub Token

Follow the steps above under "Creating a GitHub Token"

#### 4. Open Cloud Image Notepad

1. Open `index.html` in your browser
2. Fill in the configuration:
   - **GitHub Username**: Your GitHub username
   - **Repository Name**: Your repository name
   - **Folder in Repository**: "images" (or your folder name)
   - **GitHub Personal Access Token**: Paste your token

3. Click "Refresh List" to see existing images
4. Click "Add Image" to upload a new image

## 📝 Usage Examples

### Example 1: Upload Your First Image

1. Click "Select Image to Upload"
2. Choose an image from your computer
3. Click "➕ Add Image"
4. See "✅ Image uploaded successfully!" message
5. Your image appears in the gallery

### Example 2: Update an Existing Image

1. Select a new image with the **same filename** as an existing one
2. Click "➕ Add Image"
3. The app detects the existing file and updates it
4. The gallery refreshes automatically

### Example 3: Delete an Image

1. Click "🗑️ Delete" on any image card
2. Confirm the deletion
3. The image is removed from GitHub and the gallery

### Example 4: Share Images

Since images are stored in a public GitHub repository:
1. Your images are accessible via GitHub's public URL
2. You can share the GitHub repository link with others
3. They can view images directly on GitHub.com

## 🛠️ How It Works Technically

### Architecture

```
┌─────────────────────┐
│  Browser (You)      │
│  Cloud Image App    │
└──────────┬──────────┘
           │
           │ HTTPS Request with Token
           │ PUT/GET /contents/
           ↓
┌─────────────────────┐
│  GitHub API         │
│  api.github.com     │
└──────────┬──────────┘
           │
           │ Validates Token
           │ Checks Permissions
           ↓
┌─────────────────────┐
│  GitHub Server      │
│  Repository Storage │
└─────────────────────┘
```

### Image Upload Process

1. **Select Image**: User chooses image file from computer
2. **Read File**: Browser reads file using `FileReader` API
3. **Encode**: Convert image to Base64 format
4. **Create Request**: Build API request with:
   - Token in Authorization header
   - Base64 content in request body
   - File path and commit message
5. **Send to GitHub**: PUT request to GitHub API
6. **GitHub Response**: Returns success or error
7. **Refresh Gallery**: App reloads images from GitHub
8. **Display**: New image appears in gallery

### Base64 Encoding

The app uses **Base64 encoding** because:
- GitHub API requires text format for file content
- Base64 converts binary image data to text
- It's reversible (can be decoded back to original image)
- It's a standard, widely-supported format

```
Image File (binary)
↓
[Base64 Encoder]
↓
ghQKAAACAgAAAAvAABAAAA==... (text)
↓
[Send to GitHub API]
```

When you view the image, GitHub automatically decodes Base64 back to the original image.

## 🔄 API Endpoints Used

The app uses GitHub REST API v3:

### Load Images
```
GET /repos/{owner}/{repo}/contents/{folder}
Header: Authorization: token {token}
```

### Upload/Update Image
```
PUT /repos/{owner}/{repo}/contents/{path}
Header: Authorization: token {token}
Body: { message, content (Base64), sha }
```

### Delete Image
```
DELETE /repos/{owner}/{repo}/contents/{path}
Header: Authorization: token {token}
Body: { message, sha }
```

## ❌ Troubleshooting

### Error: "Repository, folder, or token not found"

**Cause**: Configuration is incorrect or token is invalid

**Solutions**:
1. Verify GitHub username matches your GitHub account
2. Check repository name is spelled correctly
3. Confirm token is valid and not expired
4. Ensure token has `repo` scope enabled
5. Generate a new token if needed

### Error: "Unauthorized. Your token may be invalid or expired"

**Cause**: Token is no longer valid

**Solutions**:
1. Check token expiration date in GitHub settings
2. Generate a new token with 90+ day expiration
3. Copy the new token and paste it into the app

### Error: "Network error"

**Cause**: Internet connection issue or GitHub is down

**Solutions**:
1. Check your internet connection
2. Wait a few minutes and try again
3. Check GitHub's status at status.github.com

### Images not uploading

**Cause**: File type or size issue

**Solutions**:
1. Ensure file is a valid image (JPG, PNG, GIF, WebP, SVG)
2. Check file size (GitHub has limits for single files)
3. Check repository permissions (token needs `repo` scope)

## 📊 File Size Limits

GitHub API limits file sizes:
- **Maximum file size**: 100 MB per file
- **Recommended**: Keep images under 50 MB for best performance
- **Tip**: Compress images before uploading for faster uploads

## 🎨 Customization

### Change Default Configuration

Edit `index.html` and modify these values:
```html
<input type="text" id="owner" value="your-username">
<input type="text" id="repo" value="your-repo-name">
<input type="text" id="folder" value="your-folder-name">
```

### Styling

The app uses CSS Grid for the image gallery. To customize:
1. Modify the `grid-template-columns` property in `.css`
2. Adjust colors by changing CSS variables
3. Change button text and icons as needed

## 📱 Browser Compatibility

- ✅ Chrome/Chromium (v90+)
- ✅ Firefox (v88+)
- ✅ Safari (v14+)
- ✅ Edge (v90+)
- ✅ Mobile browsers (iOS Safari, Chrome Android)

## 📄 License

Free to use for personal and commercial projects.

## 🤝 Support

For issues or questions:
1. Check GitHub API documentation: https://docs.github.com/en/rest
2. Verify token and repository configuration
3. Check browser console for detailed error messages (F12)

## 📚 Additional Resources

- [GitHub Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
- [GitHub REST API Docs](https://docs.github.com/en/rest)
- [Base64 Encoding](https://en.wikipedia.org/wiki/Base64)
- [FileReader API](https://developer.mozilla.org/en-US/docs/Web/API/FileReader)

---

**Made with ❤️ for cloud storage enthusiasts**
