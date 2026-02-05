# 💻 Cloud Image Notepad - Code Examples & API Reference

## Table of Contents

1. [JavaScript Functions](#javascript-functions)
2. [API Requests](#api-requests)
3. [Base64 Encoding](#base64-encoding)
4. [Error Handling](#error-handling)
5. [Configuration Examples](#configuration-examples)

---

## JavaScript Functions

### 1. Get Configuration

```javascript
function getConfig() {
  return {
    owner: document.getElementById("owner").value.trim(),
    repo: document.getElementById("repo").value.trim(),
    folder: document.getElementById("folder").value.trim(),
    token: document.getElementById("token").value.trim()
  };
}
```

**Usage**:
```javascript
const config = getConfig();
console.log(config.owner);    // "neiltambasacan"
console.log(config.token);    // "ghp_xxx..."
```

**Output**:
```javascript
{
  owner: "your-username",
  repo: "cloud-image-notepad",
  folder: "images",
  token: "ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```

### 2. Load Images from GitHub

```javascript
async function loadImages() {
  const config = getConfig();
  if (!validateConfig(config)) return;
  
  showStatus("🔄 Loading images from cloud...", 'info');
  
  try {
    const response = await fetch(
      `https://api.github.com/repos/${config.owner}/${config.repo}/contents/${config.folder}`,
      {
        headers: {
          Authorization: `token ${config.token}`,
          Accept: "application/vnd.github.v3+json"
        }
      }
    );

    if (!response.ok) {
      handleLoadError(response.status);
      return;
    }

    const files = await response.json();
    const imageFiles = files.filter(file => 
      file.type === "file" && /\.(jpg|jpeg|png|gif|webp|svg)$/i.test(file.name)
    );

    displayImages(imageFiles);
    showStatus(`✅ Loaded ${imageFiles.length} image(s)`, 'success');
  } catch (error) {
    showStatus(`❌ Error: ${error.message}`, 'error');
  }
}
```

**Step-by-Step**:
1. Get config (owner, repo, folder, token)
2. Build GitHub API URL
3. Add authorization header with token
4. Fetch from GitHub
5. Filter for image files only
6. Display in gallery
7. Show success message

### 3. Upload Image with Base64

```javascript
async function uploadImage() {
  const config = getConfig();
  if (!validateConfig(config)) return;

  const fileInput = document.getElementById("imageInput");
  const file = fileInput.files[0];

  if (!file) {
    showStatus("❌ Please select an image file", 'error');
    return;
  }

  showStatus("⏳ Uploading image...", 'info');
  const reader = new FileReader();

  reader.onload = async function () {
    try {
      // Step 1: Extract Base64 content (remove data URL prefix)
      const base64Content = reader.result.split(",")[1];
      const filePath = `${config.folder}/${file.name}`;

      // Step 2: Check if file already exists
      let sha = null;
      try {
        const checkResponse = await fetch(
          `https://api.github.com/repos/${config.owner}/${config.repo}/contents/${filePath}`,
          {
            headers: {
              Authorization: `token ${config.token}`,
              Accept: "application/vnd.github.v3+json"
            }
          }
        );
        
        if (checkResponse.ok) {
          const data = await checkResponse.json();
          sha = data.sha;  // Need SHA to update existing file
        }
      } catch (e) {
        // File doesn't exist, sha stays null
      }

      // Step 3: Upload/update file to GitHub
      const uploadResponse = await fetch(
        `https://api.github.com/repos/${config.owner}/${config.repo}/contents/${filePath}`,
        {
          method: "PUT",
          headers: {
            Authorization: `token ${config.token}`,
            "Content-Type": "application/json",
            Accept: "application/vnd.github.v3+json"
          },
          body: JSON.stringify({
            message: sha ? `Update image: ${file.name}` : `Add image: ${file.name}`,
            content: base64Content,  // Base64 encoded image
            sha: sha,                 // null for new files, sha for updates
            committer: {
              name: "Cloud Image Notepad",
              email: "app@cloud-notepad.local"
            }
          })
        }
      );

      if (!uploadResponse.ok) {
        const errorData = await uploadResponse.json();
        showStatus(`❌ Upload failed: ${errorData.message}`, 'error');
        return;
      }

      showStatus(`✅ Image uploaded successfully!`, 'success');
      fileInput.value = "";  // Clear file input
      await new Promise(r => setTimeout(r, 1000));
      loadImages();  // Refresh gallery
    } catch (error) {
      showStatus(`❌ Error: ${error.message}`, 'error');
    }
  };

  reader.readAsDataURL(file);  // Read file as Data URL (Base64)
}
```

**Upload Flow**:
```
Select Image File
        ↓
FileReader reads file
        ↓
Convert to Base64
        ↓
Check if file exists
        ↓
If exists: Get SHA (needed for update)
If new: SHA = null
        ↓
Send PUT request to GitHub
        ↓
Include Base64 content
Include SHA (if updating)
        ↓
GitHub processes request
        ↓
✅ Success: Refresh gallery
❌ Error: Show error message
```

### 4. Delete Image from Cloud

```javascript
async function deleteImage(fileName) {
  if (!confirm(`Delete "${fileName}"? This cannot be undone.`)) {
    return;
  }

  const config = getConfig();
  if (!validateConfig(config)) return;

  showStatus("⏳ Deleting image...", 'info');
  
  try {
    const filePath = `${config.folder}/${fileName}`;
    
    // Step 1: Get file details including SHA
    const getResponse = await fetch(
      `https://api.github.com/repos/${config.owner}/${config.repo}/contents/${filePath}`,
      {
        headers: {
          Authorization: `token ${config.token}`,
          Accept: "application/vnd.github.v3+json"
        }
      }
    );

    if (!getResponse.ok) {
      showStatus("❌ Failed to find file for deletion", 'error');
      return;
    }

    const fileData = await getResponse.json();

    // Step 2: Delete the file using SHA
    const deleteResponse = await fetch(
      `https://api.github.com/repos/${config.owner}/${config.repo}/contents/${filePath}`,
      {
        method: "DELETE",
        headers: {
          Authorization: `token ${config.token}`,
          "Content-Type": "application/json",
          Accept: "application/vnd.github.v3+json"
        },
        body: JSON.stringify({
          message: `Delete image: ${fileName}`,
          sha: fileData.sha,  // Required for deletion
          committer: {
            name: "Cloud Image Notepad",
            email: "app@cloud-notepad.local"
          }
        })
      }
    );

    if (!deleteResponse.ok) {
      showStatus("❌ Failed to delete image", 'error');
      return;
    }

    showStatus(`✅ Image deleted successfully!`, 'success');
    await new Promise(r => setTimeout(r, 1000));
    loadImages();  // Refresh gallery
  } catch (error) {
    showStatus(`❌ Error: ${error.message}`, 'error');
  }
}
```

**Delete Flow**:
```
User clicks Delete
        ↓
Confirm dialog
        ↓
Get file details (to retrieve SHA)
        ↓
Send DELETE request
Include SHA
        ↓
GitHub deletes file
        ↓
✅ Success: Refresh gallery
❌ Error: Show error message
```

---

## API Requests

### 1. Load Images (GET Request)

```http
GET /repos/your-username/cloud-image-notepad/contents/images HTTP/1.1
Host: api.github.com
Authorization: token ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Accept: application/vnd.github.v3+json
```

**Response (Success - 200)**:
```json
[
  {
    "name": "photo1.jpg",
    "path": "images/photo1.jpg",
    "sha": "abc123def456...",
    "size": 45678,
    "type": "file",
    "download_url": "https://raw.githubusercontent.com/your-username/cloud-image-notepad/main/images/photo1.jpg",
    "html_url": "https://github.com/your-username/cloud-image-notepad/blob/main/images/photo1.jpg"
  },
  {
    "name": "photo2.png",
    "path": "images/photo2.png",
    "sha": "xyz789abc...",
    "size": 67890,
    "type": "file",
    "download_url": "https://raw.githubusercontent.com/your-username/cloud-image-notepad/main/images/photo2.png",
    "html_url": "https://github.com/your-username/cloud-image-notepad/blob/main/images/photo2.png"
  }
]
```

**Response (Error - 401)**:
```json
{
  "message": "Bad credentials",
  "documentation_url": "https://docs.github.com/rest"
}
```

**Response (Error - 404)**:
```json
{
  "message": "Not Found",
  "documentation_url": "https://docs.github.com/rest/reference/repos#get-repository-content"
}
```

### 2. Upload Image (PUT Request)

```http
PUT /repos/your-username/cloud-image-notepad/contents/images/photo.jpg HTTP/1.1
Host: api.github.com
Authorization: token ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Content-Type: application/json
Accept: application/vnd.github.v3+json

{
  "message": "Add image: photo.jpg",
  "content": "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNk+M9QDwADhgGAWjR9awAAAABJRU5ErkJggg==",
  "committer": {
    "name": "Cloud Image Notepad",
    "email": "app@cloud-notepad.local"
  }
}
```

**Response (Success - 201)**:
```json
{
  "content": {
    "name": "photo.jpg",
    "path": "images/photo.jpg",
    "sha": "abc123def456...",
    "size": 45678,
    "type": "file",
    "download_url": "https://raw.githubusercontent.com/your-username/cloud-image-notepad/main/images/photo.jpg"
  },
  "commit": {
    "sha": "commit123abc...",
    "message": "Add image: photo.jpg",
    "author": {
      "name": "Cloud Image Notepad",
      "email": "app@cloud-notepad.local",
      "date": "2024-02-05T12:34:56Z"
    }
  }
}
```

### 3. Update Existing Image (PUT with SHA)

```http
PUT /repos/your-username/cloud-image-notepad/contents/images/photo.jpg HTTP/1.1
Host: api.github.com
Authorization: token ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Content-Type: application/json
Accept: application/vnd.github.v3+json

{
  "message": "Update image: photo.jpg",
  "content": "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNk+M9QDwADhgGAWjR9awAAAABJRU5ErkJggg==",
  "sha": "abc123def456...",
  "committer": {
    "name": "Cloud Image Notepad",
    "email": "app@cloud-notepad.local"
  }
}
```

**Key Difference**: When updating, include `"sha"` field with the current file's SHA.

### 4. Delete Image (DELETE Request)

```http
DELETE /repos/your-username/cloud-image-notepad/contents/images/photo.jpg HTTP/1.1
Host: api.github.com
Authorization: token ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Content-Type: application/json
Accept: application/vnd.github.v3+json

{
  "message": "Delete image: photo.jpg",
  "sha": "abc123def456..."
}
```

**Response (Success - 200)**:
```json
{
  "content": null,
  "commit": {
    "sha": "commit456def...",
    "message": "Delete image: photo.jpg",
    "author": {
      "name": "Cloud Image Notepad",
      "email": "app@cloud-notepad.local",
      "date": "2024-02-05T12:35:00Z"
    }
  }
}
```

---

## Base64 Encoding

### What is Base64?

Base64 is a way to convert binary data (images) into text that can be transmitted over the internet.

### Encoding Example

```javascript
// Original image file (binary)
File: photo.jpg (45,678 bytes)

// Read as Data URL
reader.readAsDataURL(file);
// Result: "data:image/jpeg;base64,/9j/4AAQSkZJRg..."

// Extract Base64 part
const base64 = dataUrl.split(",")[1];
// Result: "/9j/4AAQSkZJRg..."

// Send to GitHub in request body
{
  "content": "/9j/4AAQSkZJRg..."
}

// GitHub stores as Base64
// When you access via raw.githubusercontent.com, GitHub decodes it back to image
```

### How It Works

```
Image Bytes (Binary)
  0x89 0x50 0x4E 0x47 ...
        ↓
  [Base64 Encoder]
        ↓
Text Characters
  /9j/4AAQSkZJRg...
        ↓
Transmit over Internet
        ↓
GitHub stores
        ↓
GitHub decodes back to binary
        ↓
Browser displays as image
```

### JavaScript Code

```javascript
// Read file as Base64
const reader = new FileReader();

reader.onload = function() {
  // reader.result = "data:image/jpeg;base64,/9j/4AAQSkZJRg..."
  const base64Content = reader.result.split(",")[1];
  // base64Content = "/9j/4AAQSkZJRg..."
  
  // Send in API request
  body: JSON.stringify({
    content: base64Content
  })
};

reader.readAsDataURL(file);
```

### Size Increase from Base64

Base64 makes data ~33% larger:

```
Original JPEG: 45,678 bytes
Base64 encoded: ~60,904 bytes (33% larger)

Original PNG: 67,890 bytes
Base64 encoded: ~90,520 bytes (33% larger)
```

**Why this is okay**: 
- GitHub supports up to 100 MB files
- HTTPS compression helps reduce transmitted size
- The overhead is acceptable for most images

---

## Error Handling

### HTTP Status Codes

```javascript
// 200 OK - Success
if (response.status === 200) {
  // Request succeeded
  const data = await response.json();
  console.log("Success:", data);
}

// 201 Created - File created successfully
if (response.status === 201) {
  // File was created (upload successful)
  const data = await response.json();
  console.log("File created:", data);
}

// 401 Unauthorized - Invalid token
if (response.status === 401) {
  showStatus("❌ Unauthorized. Token invalid or expired.", 'error');
}

// 403 Forbidden - Token doesn't have permission
if (response.status === 403) {
  showStatus("❌ Forbidden. Token scope insufficient.", 'error');
}

// 404 Not Found - Repository/folder/file not found
if (response.status === 404) {
  showStatus("❌ Repository or folder not found.", 'error');
}

// 422 Unprocessable Entity - Invalid request body
if (response.status === 422) {
  showStatus("❌ Invalid request. Check file format.", 'error');
}

// 500+ Server Error - GitHub server issue
if (response.status >= 500) {
  showStatus("❌ GitHub server error. Try again later.", 'error');
}
```

### Common Errors

#### Error 1: Invalid Token

```
Response: 401 Unauthorized
Message: "Bad credentials"

Cause:
  - Token is expired
  - Token is invalid/malformed
  - Token was deleted

Solution:
  1. Check GitHub Settings → Tokens
  2. Verify token expiration date
  3. Generate new token if needed
  4. Paste new token into app
```

#### Error 2: Repository Not Found

```
Response: 404 Not Found
Message: "Not Found"

Cause:
  - Username is wrong
  - Repository name is wrong
  - Repository was deleted
  - Repository is private (but token lacks access)

Solution:
  1. Verify GitHub username
  2. Check repository name spelling
  3. Ensure token has repo scope
  4. Verify user can access repo
```

#### Error 3: Insufficient Permissions

```
Response: 403 Forbidden
Message: "Resource not accessible by integration"

Cause:
  - Token doesn't have repo scope
  - User doesn't have push access
  - Repository is archived

Solution:
  1. Check GitHub Settings → Tokens
  2. Verify repo scope is checked
  3. Confirm user can push to repo
  4. Generate new token with correct scopes
```

#### Error 4: Network Error

```
Error: NetworkError or TypeError
Message: "Failed to fetch"

Cause:
  - No internet connection
  - GitHub is down
  - CORS issue
  - Request timeout

Solution:
  1. Check internet connection
  2. Check github.com status
  3. Try again after a few seconds
  4. Check browser console (F12)
```

### Error Handling Example

```javascript
try {
  const response = await fetch(url, options);

  // Handle HTTP error status
  if (!response.ok) {
    if (response.status === 401) {
      throw new Error("Invalid or expired token");
    } else if (response.status === 404) {
      throw new Error("Repository or folder not found");
    } else if (response.status === 403) {
      throw new Error("Insufficient permissions");
    } else {
      throw new Error(`HTTP ${response.status} error`);
    }
  }

  const data = await response.json();
  // Process successful response
  console.log("Success:", data);

} catch (error) {
  // Handle network errors
  if (error instanceof TypeError) {
    console.error("Network error:", error.message);
    showStatus("❌ Network error. Check connection.", 'error');
  } else {
    // Handle custom errors
    console.error("Error:", error.message);
    showStatus(`❌ Error: ${error.message}`, 'error');
  }
}
```

---

## Configuration Examples

### Example 1: Using Public Repository

```javascript
const config = {
  owner: "my-username",
  repo: "my-photos",
  folder: "images",
  token: "ghp_1234567890..." // Token still needed for API rate limits
};
```

Benefits:
- Images accessible publicly
- Can share repository link
- No need for others to use token

Risks:
- Images are public on GitHub

### Example 2: Using Private Repository

```javascript
const config = {
  owner: "my-username",
  repo: "my-private-photos",
  folder: "images",
  token: "ghp_1234567890..."
};
```

Benefits:
- Only you can see images
- Only with token can access
- More secure

Risks:
- Need to share token if others want access
- Images not discoverable

### Example 3: Multiple Folders

Create multiple tokens/apps for different folders:

```javascript
// App 1 - Personal Photos
const config1 = {
  owner: "my-username",
  repo: "my-photos",
  folder: "family",
  token: "ghp_personal..."
};

// App 2 - Work Projects
const config2 = {
  owner: "my-username",
  repo: "work-assets",
  folder: "screenshots",
  token: "ghp_work..."
};
```

Benefits:
- Separate tokens for different use cases
- Can revoke one without affecting other
- Different expiration dates
- Better security isolation

### Example 4: Organization Repository

```javascript
const config = {
  owner: "my-organization",  // Organization name, not personal
  repo: "shared-assets",
  folder: "images",
  token: "ghp_org_token..." // Must have access to organization repos
};
```

Requirements:
- Token must have `repo` scope for organization
- User must be member of organization
- User must have push access to repository

---

## Testing the Application

### Manual Testing Steps

1. **Test Configuration Loading**
   ```javascript
   // Open browser console (F12)
   getConfig()
   // Should show { owner, repo, folder, token }
   ```

2. **Test Image Loading**
   - Enter valid configuration
   - Click "Refresh List"
   - Check console for network request
   - Verify images appear (or empty state)

3. **Test Image Upload**
   - Select an image
   - Click "Add Image"
   - Check console for PUT request
   - Verify "Success" message
   - Verify image appears in gallery

4. **Test Error Handling**
   - Enter invalid token
   - Click "Refresh List"
   - Check error message appears
   - Try with valid token again

5. **Test Token Expiration**
   - Use expired token
   - Click any button
   - Verify "401 Unauthorized" error
   - Generate new token
   - Test again

### Browser Console Tips

```javascript
// See all requests
// Open DevTools → Network tab → Filter "api.github.com"

// Check configuration
getConfig()

// Test API directly
fetch("https://api.github.com/repos/owner/repo/contents/folder", {
  headers: {
    Authorization: "token ghp_xxx...",
    Accept: "application/vnd.github.v3+json"
  }
}).then(r => r.json()).then(d => console.log(d))

// Monitor status messages
document.getElementById("status")

// Check localStorage
localStorage.getItem("imageNotepad_owner")
```

---

## Performance Optimization Tips

1. **Image Compression**
   ```
   - Compress images before upload
   - Target: < 50 MB per image
   - Tools: ImageOptim, TinyPNG, etc.
   ```

2. **Limit Folder Size**
   ```
   - Keep under 1,000 images per folder
   - Better user experience with pagination
   - Faster loading and refreshing
   ```

3. **Lazy Loading**
   ```
   - Add loading="lazy" to img tags
   - Load images only when visible
   - Improves initial page load
   ```

4. **Batch Operations**
   ```
   - Upload multiple images in sequence
   - Not simultaneously (to avoid rate limits)
   - Better error handling
   ```

---

This completes the technical documentation for Cloud Image Notepad! 🚀
