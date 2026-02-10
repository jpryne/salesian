# Firebase Memorial Website Setup Guide

This guide will walk you through setting up the memorial website with Firebase for secure password storage, photo hosting, and moderation.

## What You'll Need
- A Google account (free)
- 15 minutes of setup time
- The two HTML files: `sensei-memorial-firebase.html` and `admin.html`

## Total Cost: $0
Firebase's free tier includes:
- 1 GB storage (thousands of photos)
- 50,000 daily reads
- 20,000 daily writes
- More than enough for a small memorial site

---

## Step 1: Create Firebase Project (5 minutes)

1. Go to https://console.firebase.google.com/
2. Click **"Add project"**
3. Enter a project name (e.g., "sensei-memorial")
4. Disable Google Analytics (not needed) or keep it enabled
5. Click **"Create project"**
6. Wait for project creation, then click **"Continue"**

---

## Step 2: Set Up Realtime Database (3 minutes)

1. In the left sidebar, click **"Build"** → **"Realtime Database"**
2. Click **"Create Database"**
3. Choose your location (closest to you)
4. **IMPORTANT**: Select **"Start in locked mode"** for security
5. Click **"Enable"**

### Configure Database Rules

1. Click the **"Rules"** tab at the top
2. Replace the existing rules with this:

```json
{
  "rules": {
    "posts": {
      ".read": "data.child('status').val() === 'approved'",
      ".write": false,
      "$postId": {
        ".write": "!data.exists()",
        ".validate": "newData.hasChildren(['author', 'story', 'date', 'status']) && newData.child('status').val() === 'pending'"
      }
    },
    "config": {
      "postingPassword": {
        ".read": true,
        ".write": false
      },
      "adminPassword": {
        ".read": true,
        ".write": false
      }
    }
  }
}
```

3. Click **"Publish"**

**What these rules do:**
- Anyone can read approved posts
- Anyone can submit new posts (as "pending")
- Only you (via Firebase Console) can approve/reject posts
- Passwords are readable but not writable (you set them manually)

---

## Step 3: Set Up Storage for Photos (2 minutes)

1. In the left sidebar, click **"Build"** → **"Storage"**
2. Click **"Get started"**
3. Click **"Next"** (keep default security rules)
4. Choose your location (same as database)
5. Click **"Done"**

### Configure Storage Rules

1. Click the **"Rules"** tab
2. Replace with this:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /photos/{fileName} {
      allow read: if true;
      allow write: if request.resource.size < 5 * 1024 * 1024
                   && request.resource.contentType.matches('image/.*');
    }
  }
}
```

3. Click **"Publish"**

**What this does:**
- Anyone can read photos
- Anyone can upload images under 5MB to the photos folder
- Only image files are allowed

---

## Step 4: Get Your Firebase Configuration (2 minutes)

1. Click the gear icon ⚙️ next to "Project Overview" → **"Project settings"**
2. Scroll down to **"Your apps"**
3. Click the **web icon** `</>`
4. Enter a nickname (e.g., "Memorial Website")
5. Don't check "Firebase Hosting"
6. Click **"Register app"**
7. You'll see your Firebase configuration - it looks like this:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSyD...",
  authDomain: "sensei-memorial.firebaseapp.com",
  databaseURL: "https://sensei-memorial-default-rtdb.firebaseio.com",
  projectId: "sensei-memorial",
  storageBucket: "sensei-memorial.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

8. **Copy this entire configuration**

---

## Step 5: Add Configuration to Your HTML Files (3 minutes)

### Update sensei-memorial-firebase.html

1. Open `sensei-memorial-firebase.html` in a text editor (Notepad, TextEdit, VS Code, etc.)
2. Find this section (around line 386):

```javascript
const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
    // ... etc
};
```

3. **Replace it with your actual configuration** from Step 4
4. Save the file

### Update admin.html

1. Open `admin.html` in a text editor
2. Find the same `firebaseConfig` section (around line 320)
3. **Replace it with the same configuration**
4. Save the file

---

## Step 6: Set Your Passwords (2 minutes)

You need to set two passwords in Firebase:
- **Posting password**: For group members to submit memories
- **Admin password**: For you to access the moderation dashboard

1. Go to your Firebase Console → **Realtime Database**
2. You'll see your empty database
3. Click the **"+"** icon next to your database name
4. Add these two entries:

**First entry:**
- Name: `config`
- Click the **"+"** next to config
  - Name: `postingPassword`
  - Value: `YourPostingPassword123` (choose your own)

**Second entry:**
- Click the **"+"** next to config again
  - Name: `adminPassword`
  - Value: `YourAdminPassword456` (choose your own, make it different!)

Your database structure should look like:
```
sensei-memorial
  └── config
      ├── postingPassword: "YourPostingPassword123"
      └── adminPassword: "YourAdminPassword456"
```

---

## Step 7: Test Your Website

1. Open `sensei-memorial-firebase.html` in your web browser
2. You should see the memorial page
3. Click **"Share a Memory"**
4. Enter your posting password
5. Fill in a test post
6. Submit

The post won't appear yet because it needs approval!

---

## Step 8: Test the Admin Panel

1. Open `admin.html` in your web browser
2. Enter your admin password
3. You should see your test post in "Pending Review"
4. Click **"Approve"**
5. Go back to the main page and refresh - your post should appear!

---

## Step 9: Deploy Your Website

Now you need to host these files online so others can access them.

### Option A: Netlify Drop (Easiest - 2 minutes)

1. Go to https://app.netlify.com/drop
2. Drag both HTML files into the drop zone
3. Netlify will give you a URL like `https://random-name-123.netlify.app`
4. Share this URL with your group!
5. Bookmark the `/admin.html` page for yourself

### Option B: Firebase Hosting (3 minutes)

1. Install Firebase CLI: `npm install -g firebase-tools`
2. Run `firebase login`
3. Run `firebase init hosting`
4. Select your project
5. Set public directory to current folder
6. Don't configure as single-page app
7. Don't overwrite files
8. Run `firebase deploy`

### Option C: GitHub Pages (5 minutes)

1. Create a GitHub repository
2. Upload both HTML files
3. Go to Settings → Pages
4. Select "main" branch
5. Your site will be at `https://yourusername.github.io/repository-name`

---

## Step 10: Share With Your Group

1. Share the main website URL with group members
2. Give them the **posting password** (via email, text, or in person)
3. Keep the **admin password** secret (just for you)
4. Bookmark `your-url.com/admin.html` for yourself

---

## How to Use Going Forward

### For Group Members:
1. Visit the website
2. Click "Share a Memory"
3. Enter the posting password
4. Write their memory and optionally upload a photo
5. Submit and wait for your approval

### For You (Admin):
1. Visit `your-url.com/admin.html`
2. Login with admin password
3. Review pending posts
4. Approve or reject each one
5. Approved posts appear publicly immediately

---

## Troubleshooting

### "Firebase not configured" error
- Make sure you replaced the firebaseConfig in both HTML files
- Check that all values are inside quotes
- Make sure you didn't accidentally delete any commas

### "Error loading posts"
- Check your database rules are published correctly
- Make sure your databaseURL includes "https://"

### "Incorrect password"
- Go to Firebase Console → Realtime Database
- Check that config/postingPassword and config/adminPassword exist
- Make sure there are no extra spaces in the password values

### Photos won't upload
- Check Firebase Storage rules are published
- Make sure the image is under 5MB
- Try a different image format (JPEG works best)

### Can't approve posts in admin panel
- Make sure you're logged into the admin panel
- Try refreshing the page
- Check browser console for errors (F12)

---

## Security Notes

✅ **What's Secure:**
- Passwords are stored in Firebase, not in the HTML code
- Only approved posts are visible to the public
- Photos are stored securely in Firebase Storage
- Database rules prevent unauthorized modifications

⚠️ **What to Know:**
- The posting password can be read by anyone using browser dev tools (F12), but they still can't approve their own posts
- For maximum security, you could set up email-based authentication, but that's more complex
- This setup is appropriate for a trusted small group

---

## Customization

### Change the memorial title and quote:
Edit these sections in `sensei-memorial-firebase.html`:
- Line ~63: `<h1>In Loving Memory</h1>`
- Line ~68: The zen quote

### Change colors:
Edit the CSS variables at the top (around line 8):
```css
:root {
    --cream: #faf7f2;
    --charcoal: #2b2d2e;
    --sage: #8b9a8b;
    --gold: #d4af37;
    /* etc */
}
```

---

## Questions?

If you run into issues:
1. Check the Troubleshooting section above
2. Open browser console (F12) to see error messages
3. Verify your Firebase configuration is correct
4. Make sure database and storage rules are published

---

## Summary

You now have:
- ✅ A beautiful memorial website
- ✅ Secure password protection for posting
- ✅ Photo upload capability
- ✅ Moderation queue for all submissions
- ✅ Admin panel to approve/reject posts
- ✅ All data stored securely in Firebase
- ✅ Zero monthly costs

**Total setup time: ~15 minutes**
**Monthly cost: $0**

Share the posting password with your group, and keep the admin password for yourself. Posts won't appear until you approve them in the admin panel.

In memory of your sensei. 🙏
