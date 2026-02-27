# CIVITAS - Deployment Instructions

## Files Included
- `index.html` - Main application
- `netlify.toml` - Netlify deployment config
- `database-rules.json` - Firebase security rules (NOT test mode)

---

## Step 1: Create Firebase Project

1. Go to https://console.firebase.google.com
2. Click **"Create a project"**
3. Name it (e.g., "civitas-classroom")
4. Disable Google Analytics (not needed) → **Create Project**

---

## Step 2: Create Realtime Database

1. In Firebase console, click **"Build"** → **"Realtime Database"**
2. Click **"Create Database"**
3. Choose your region (us-central1 is fine)
4. Select **"Start in locked mode"** ← Important: NOT test mode!
5. Click **"Enable"**

---

## Step 3: Set Security Rules

1. In Realtime Database, click the **"Rules"** tab
2. Delete everything and paste this:

```json
{
  "rules": {
    "sessions": {
      "$sessionCode": {
        ".read": true,
        ".write": true,
        ".validate": "newData.hasChildren(['teacher', 'created', 'status'])",
        
        "teacher": {
          ".validate": "newData.isString() && newData.val().length > 0"
        },
        "created": {
          ".validate": "newData.isNumber()"
        },
        "status": {
          ".validate": "newData.isString() && (newData.val() == 'waiting' || newData.val() == 'playing' || newData.val() == 'ended')"
        },
        "students": {
          "$studentId": {
            ".validate": "newData.hasChildren(['name', 'joined'])",
            "name": {
              ".validate": "newData.isString() && newData.val().length >= 1 && newData.val().length <= 20"
            },
            "joined": {
              ".validate": "newData.isNumber()"
            }
          }
        }
      }
    },
    
    ".read": false,
    ".write": false
  }
}
```

3. Click **"Publish"**

---

## Step 4: Get Firebase Config

1. Click the **gear icon ⚙️** → **"Project settings"**
2. Scroll down to "Your apps"
3. Click **"</>"** (Web) to add a web app
4. Name it anything (e.g., "civitas-web")
5. DON'T check "Firebase Hosting"
6. Click **"Register app"**
7. You'll see config code - copy the object that looks like:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "civitas-classroom.firebaseapp.com",
  databaseURL: "https://civitas-classroom-default-rtdb.firebaseio.com",
  projectId: "civitas-classroom",
  storageBucket: "civitas-classroom.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

---

## Step 5: Add Config to index.html

1. Open `index.html` in a text editor
2. Find this section near the bottom (around line 220):

```javascript
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  ...
};
```

3. Replace it with YOUR config from Step 4
4. Save the file

---

## Step 6: Deploy to Netlify

### Option A: Drag & Drop
1. Go to https://app.netlify.com
2. Drag the entire folder containing these files
3. Done! You'll get a URL like `random-name.netlify.app`

### Option B: Connect GitHub
1. Push these files to a GitHub repo
2. In Netlify, click **"Add new site"** → **"Import existing project"**
3. Connect your GitHub repo
4. Build settings should auto-detect from `netlify.toml`
5. Click **"Deploy"**

---

## Usage

### Teacher:
1. Open your Netlify URL
2. Click **"Teacher"**
3. Enter authorized email
4. Click **"Copy Student Link"**
5. Share link with students
6. Click **"Start Game"** when ready

### Students:
1. Click the link from teacher
2. Enter first name
3. Click **"Join Game"**
4. Wait for teacher to start

---

## Security Notes

These rules are secure because:
- Only `/sessions` path is accessible (not entire database)
- Data must have correct structure (validated)
- Only valid session codes work
- No authentication = no passwords to manage
- Sessions are temporary (teacher deletes when done)

For a classroom game, this is appropriate security.
