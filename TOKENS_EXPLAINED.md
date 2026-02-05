# 🔐 GitHub Tokens & Authentication - Detailed Explanation

## Table of Contents

1. [What Are Tokens?](#what-are-tokens)
2. [How Tokens Work](#how-tokens-work)
3. [Token Creation Process](#token-creation-process)
4. [Token Security](#token-security)
5. [Token Expiration](#token-expiration)
6. [API Authentication Flow](#api-authentication-flow)
7. [Scope Permissions](#scope-permissions)
8. [Token vs Password](#token-vs-password)

---

## What Are Tokens?

A **GitHub Personal Access Token (PAT)** is a unique string of characters that serves as an authentication credential.

```
Example Token Format: ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Length: 36+ characters
Type: Secure credential similar to a password
```

### Key Characteristics

| Feature | Token | Password |
|---------|-------|----------|
| Expiration | Can set (90+ days) | No expiration |
| Scope | Limited permissions | Full access |
| Revocability | Instant | Manual GitHub logout required |
| Visibility | One-time display | Only known to you |
| Reusability | For one app typically | For all GitHub access |

---

## How Tokens Work

### The Authentication Process

```
┌──────────────────────────────────────────────────────┐
│ Step 1: User Authenticates to GitHub                │
│ → Goes to github.com                                 │
│ → Enters username + password                         │
│ → GitHub verifies credentials                        │
│ → User is logged in                                  │
└────────────┬─────────────────────────────────────────┘
             │
┌────────────▼─────────────────────────────────────────┐
│ Step 2: User Creates Personal Access Token           │
│ → Settings → Developer Settings → Tokens             │
│ → Click "Generate new token"                         │
│ → Set expiration date (90+ days)                     │
│ → Select scopes (e.g., repo, read:user)            │
│ → GitHub generates random 36-char string            │
│ → Shows token ONCE (never again!)                   │
└────────────┬─────────────────────────────────────────┘
             │
┌────────────▼─────────────────────────────────────────┐
│ Step 3: User Copies Token to App                     │
│ → Copies: ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxx           │
│ → Pastes into Cloud Image Notepad app               │
│ → App stores in browser memory or localStorage      │
└────────────┬─────────────────────────────────────────┘
             │
┌────────────▼─────────────────────────────────────────┐
│ Step 4: App Makes API Request with Token            │
│ → App reads image file                               │
│ → Encodes to Base64                                  │
│ → Sends PUT request to GitHub API with:            │
│    - Header: Authorization: token ghp_xxx...       │
│    - Body: base64 image content                     │
│ → HTTPS encrypted transmission                      │
└────────────┬─────────────────────────────────────────┘
             │
┌────────────▼─────────────────────────────────────────┐
│ Step 5: GitHub Validates Token                       │
│ → Checks if token exists in database                │
│ → Verifies token hasn't expired                     │
│ → Checks token scopes (repo, etc)                   │
│ → Verifies associated user has repo access         │
└────────────┬─────────────────────────────────────────┘
             │
        ┌────┴────┐
        │          │
        ▼          ▼
    ✅ VALID    ❌ INVALID
        │          │
    ┌───┴──┐   ┌───┴──┐
    │      │   │      │
    ▼      ▼   ▼      ▼
  Allow  Update Error  401
  Request File  Code  Unauthorized
```

---

## Token Creation Process

### Detailed Steps

#### 1. Navigate to GitHub Settings

**URL**: https://github.com/settings/tokens

Or manually:
1. Click profile icon (top-right)
2. Click "Settings"
3. Scroll to "Developer settings"
4. Click "Personal access tokens"
5. Click "Tokens (classic)"

#### 2. Generate New Token

Click **"Generate new token (classic)"** button

#### 3. Configure Token

```
┌─────────────────────────────────────┐
│ New Personal Access Token           │
├─────────────────────────────────────┤
│ Token name: *                       │
│ [Cloud Image Notepad__________] ◄── Your token name
│                                     │
│ Expiration:                         │
│ ○ No expiration                     │
│ ○ 7 days                            │
│ ○ 30 days                           │
│ ● 90 days ◄────────────────────────── SET THIS
│ ○ Custom...                         │
│                                     │
│ Select scopes:                      │
│ ☐ repo ◄───────────────────────── CHECK THIS
│   └─ repo:status                    │
│   └─ repo_deployment                │
│   └─ public_repo                    │
│   └─ repo:invite                    │
│ ☐ workflow                          │
│ ☐ gist                              │
│ ☐ read:user                         │
│ ... (other scopes)                  │
│                                     │
│           [Generate token]          │
└─────────────────────────────────────┘
```

#### 4. Copy Token Immediately

After clicking "Generate token":

```
┌──────────────────────────────────────────┐
│ ✓ Personal Access Token Created          │
├──────────────────────────────────────────┤
│ ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx  │
│                                          │
│ ⚠️ IMPORTANT: Make sure to copy your    │
│    token now. You won't be able to see  │
│    it again!                            │
│                                          │
│ [Copy]  [Done]                           │
└──────────────────────────────────────────┘
```

**⚠️ CRITICAL**: 
- Copy token immediately
- GitHub shows it ONLY ONCE
- If you don't copy it, you must delete and create a new one

---

## Token Security

### 🔒 Security Features

1. **Unique per Token**
   - Each token is randomly generated
   - Cryptographically secure randomness
   - 36+ characters = Extremely hard to guess
   
2. **Time-Limited**
   - Expires after set period (90+ days recommended)
   - No perpetual access
   - Reduces window of exposure if leaked

3. **Scope-Limited**
   - Only grants specific permissions
   - `repo` scope = read/write repository content
   - Doesn't grant delete repo, admin access, etc.

4. **One-Time Display**
   - Can't be viewed again after creation
   - Encourages secure storage practices
   - Prevents accidental exposure

### 🚨 Security Threats

#### Threat 1: Token Leak in Public Repository

```
❌ BAD - NEVER DO THIS:
const TOKEN = "ghp_xxxxxxxxxxxxxx"; // In code!

✅ GOOD - DO THIS:
// Token entered by user via UI
const TOKEN = document.getElementById("token").value;
```

**Risk**: Anyone can clone repo and see token

**Prevention**: 
- Keep tokens out of version control
- Use `.gitignore` for config files
- Enter tokens manually in UI

#### Threat 2: Token in Environment Variables

```
❌ .env (version controlled)
GITHUB_TOKEN=ghp_xxxxx

✅ .env.local (in .gitignore)
GITHUB_TOKEN=ghp_xxxxx
```

#### Threat 3: Token in Browser History

```
❌ Visible in URL
https://app.com/upload?token=ghp_xxxxx

✅ Hidden in POST request body
fetch('/upload', {
  method: 'POST',
  body: JSON.stringify({ token: 'ghp_xxxxx' })
})
```

**This App's Approach**: 
- Token entered in form input
- Stored in browser memory (not saved if you refresh)
- Optional localStorage storage (user controls)
- Never sent in URLs
- HTTPS encrypted transmission

### 🛡️ If Token is Compromised

1. **Immediate Actions**
   ```
   1. Go to GitHub Settings → Tokens
   2. Find the compromised token
   3. Click "Delete"
   4. Generate new token
   5. Update app with new token
   ```

2. **What the Attacker Can Do**
   - Access your images
   - Upload/modify/delete images
   - ❌ Cannot delete repository (needs admin scope)
   - ❌ Cannot access private repos they don't have access to

3. **What You Should Do**
   - Change your GitHub password (if you used weak security)
   - Check GitHub activity logs
   - Review recently accessed repositories
   - Delete token within seconds

---

## Token Expiration

### Expiration Mechanics

```
Timeline:
┌─────────┬──────────────────────────┬──────────┐
│ Day 0   │ Days 1-90                │ Day 91+  │
│         │                          │          │
│ Token   │ Token ACTIVE             │ Token    │
│ Created │ ✅ All API calls work    │ EXPIRED  │
│         │ ✅ Can read/write images │ ❌ API   │
│         │ ✅ Can delete images     │    calls │
│         │                          │    fail  │
└─────────┴──────────────────────────┴──────────┘
```

### What Happens When Token Expires

**API Response**:
```json
{
  "message": "Bad credentials",
  "documentation_url": "https://docs.github.com/rest"
}
HTTP Status: 401 Unauthorized
```

**App Shows**:
```
❌ Unauthorized. Your token may be invalid or expired.
```

### How to Renew Expired Token

1. Go to GitHub Settings → Tokens → Tokens (classic)
2. Find your token (look at "Last used" date)
3. Click "Delete" to remove expired token
4. Click "Generate new token (classic)"
5. Repeat configuration:
   - Name: "Cloud Image Notepad"
   - Expiration: 90 days
   - Scope: `repo`
6. Copy new token
7. Paste into app
8. Done! ✅

### Token Expiration Options

```
┌─────────────┬────────────────────────────────┐
│ Expiration  │ Best For                       │
├─────────────┼────────────────────────────────┤
│ 7 days      │ Testing, temporary access      │
│ 30 days     │ Short-term projects            │
│ 90 days     │ Regular use (RECOMMENDED)      │
│ Custom      │ Specific needs                 │
│ No expiry   │ ❌ NOT RECOMMENDED (risky)    │
└─────────────┴────────────────────────────────┘
```

---

## API Authentication Flow

### Request Headers with Token

Every API request includes authentication:

```http
PUT /repos/your-username/cloud-image-notepad/contents/images/photo.jpg HTTP/1.1
Host: api.github.com
Authorization: token ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Content-Type: application/json

{
  "message": "Add image: photo.jpg",
  "content": "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNk+M9QDwADhgGAWjR9awAAAABJRU5ErkJggg==",
  "committer": {
    "name": "Cloud Image Notepad",
    "email": "app@cloud-notepad.local"
  }
}
```

### Token Location in Request

```
Authorization: token {token}
                ↑      ↑
               Prefix  Your Token
```

- **Prefix**: `token` (literal word)
- **Token**: Your 36+ character ghp_xxx... string

### How GitHub Validates

```
1. Parse Authorization header
   └─ Extract: "token ghp_xxx..."

2. Look up token in database
   └─ Search for exact match

3. Check if token exists
   └─ If yes → go to step 4
   └─ If no → return 401 Unauthorized

4. Check if token expired
   └─ Compare current time to expiration
   └─ If expired → return 401 Unauthorized
   └─ If valid → go to step 5

5. Check token scopes
   └─ `repo` scope grants repository access
   └─ If scope missing → return 403 Forbidden

6. Check user permissions
   └─ Verify user can access repository
   └─ If access denied → return 403 Forbidden

7. Process request
   └─ ✅ Upload/delete/read file
   └─ ✅ Return success response
```

---

## Scope Permissions

### Understanding Scopes

A **scope** defines what the token can do.

### Available Scopes

```
repo
├─ repo:status        - Read/write repository status
├─ repo_deployment    - Manage deployments
├─ public_repo        - Access public repositories
├─ repo:invite        - Manage invitations
└─ security_events    - Read/write security events

workflow
└─ Manage GitHub Actions workflows

gist
└─ Create/edit gists

read:user
└─ Read user profile data

admin:org_hook
└─ Manage organization webhooks

admin:repo_hook
└─ Manage repository webhooks

delete_repo
└─ Delete repositories

... (30+ more scopes)
```

### This App's Requirements

```
SCOPE NEEDED: repo
PERMISSIONS GRANTED:
  ✅ Read repository contents
  ✅ Write repository contents (upload files)
  ✅ Commit changes
  ✅ Manage repository webhooks
  
PERMISSIONS NOT GRANTED:
  ❌ Delete repository
  ❌ Transfer repository
  ❌ Change repository settings
  ❌ Manage collaborators
  ❌ Access organization data
  ❌ Manage GitHub Actions
```

### Why Only `repo` Scope?

**Principle of Least Privilege**: Grant only the minimum permissions needed.

```
❌ BAD: admin scope
   └─ Can delete entire repository
   └─ Can change all settings
   └─ Can remove collaborators
   └─ Huge security risk

✅ GOOD: repo scope
   └─ Can only read/write files
   └─ Can create commits
   └─ Cannot delete repository
   └─ Cannot change settings
   └─ Limited attack surface
```

---

## Token vs Password

### Comparison

| Aspect | Token | Password |
|--------|-------|----------|
| **Length** | 36+ chars (random) | User-defined |
| **Strength** | Cryptographically strong | May be weak |
| **Expiration** | Configurable (90+ days) | Permanent |
| **Scope** | Limited permissions | Full access |
| **Revocation** | Individual tokens | Affects all sessions |
| **Storage** | In browser/app | In browser/app |
| **Best Practice** | Use for apps | Use for human login |
| **Recovery** | Generate new token | Change password |
| **Visibility** | One-time only | Never shown after set |

### Why Use Token Instead of Password?

1. **Password Safety**
   ```
   If token leaked: Delete token, make new one
   If password leaked: Change password for entire account
   
   Token is SAFER ✅
   ```

2. **Scope Control**
   ```
   Token: Access only images in one repository
   Password: Access everything on account
   
   Token is MORE RESTRICTED ✅
   ```

3. **Expiration**
   ```
   Token: Automatically expires (90 days)
   Password: Never expires unless manually changed
   
   Token is TIME-LIMITED ✅
   ```

4. **Audit Trail**
   ```
   Token: Can see last used time in GitHub
   Password: No visibility into when/where used
   
   Token is MORE TRACKABLE ✅
   ```

---

## Security Checklist

Before using Cloud Image Notepad with a token:

- [ ] Token created with 90+ day expiration
- [ ] Token has ONLY `repo` scope
- [ ] Token is NOT shared with anyone
- [ ] Token is NOT in version control (git)
- [ ] Token is NOT in browser localStorage permanently
- [ ] HTTPS is used (not HTTP)
- [ ] Browser is up-to-date with security patches
- [ ] Computer has antivirus/malware protection
- [ ] GitHub account has 2FA enabled (optional but recommended)

---

## Troubleshooting

### Issue: "401 Unauthorized"

```
Possible Causes:
1. Token is invalid
2. Token is expired
3. Token format is wrong
4. Extra spaces in token

Solutions:
1. Check token expiration in GitHub settings
2. Generate new token if expired
3. Copy token exactly (no spaces)
4. Verify Authorization header format
```

### Issue: "403 Forbidden"

```
Possible Causes:
1. Token doesn't have repo scope
2. User doesn't have repository access
3. Repository doesn't exist

Solutions:
1. Check scopes in GitHub settings
2. Verify repository name and owner
3. Check user can access repository
```

### Issue: "404 Not Found"

```
Possible Causes:
1. Repository doesn't exist
2. Username is wrong
3. Folder doesn't exist yet

Solutions:
1. Create repository if missing
2. Check username spelling
3. Upload a file to create folder
```

---

## Summary

| Concept | Key Takeaway |
|---------|--------------|
| **Token** | Secure credential for API access |
| **Expiration** | Limits exposure window (90+ days) |
| **Scope** | Restricts permissions (`repo` only) |
| **Security** | Much safer than password |
| **Usage** | Include in API request header |
| **Revocation** | Delete in GitHub settings instantly |
| **Best Practice** | Keep token secret, use HTTPS |

---

**For more information**:
- [GitHub Auth Docs](https://docs.github.com/en/authentication)
- [REST API Documentation](https://docs.github.com/en/rest)
- [Security Best Practices](https://docs.github.com/en/code-security)
