# Firestore Rules
```txt
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null && request.auth.uid == userId;
      allow update: if request.auth != null && request.auth.uid == userId;
      allow delete: if false;

      match /friends/{friendId} {
        allow read, delete: if request.auth != null && request.auth.uid == userId;
        allow create, update: if request.auth != null
          && (request.auth.uid == userId || request.auth.uid == friendId);
      }

      match /friendRequests/{requesterUid} {
        allow read, delete: if request.auth != null && request.auth.uid == userId;
        allow create: if request.auth != null && request.auth.uid == requesterUid;
        allow update: if false;
      }
    }

    match /conversations/{chatId}/messages/{messageId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null
        && request.resource.data.senderUid == request.auth.uid
        && request.resource.data.createdAt == request.time;
      allow delete: if request.auth != null
        && resource.data.senderUid == request.auth.uid;
      allow update: if false;
    }
  }
}
```

# Storage Rules
```txt
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /chat_media/{chatId}/{uid}/{fileName} {
      allow read: if request.auth != null;
      allow write: if request.auth != null
        && request.auth.uid == uid
        && request.resource.size < 26214400;
    }
  }
}
```

# Required Firebase Console Setup
1. Authentication: Enable `Email/Password` sign-in provider.
2. Authentication settings:
   - Add your domain in Authorized domains.
   - For local tests, use Firebase Emulator or add localhost.
3. Firestore: Create database in production mode, then paste rules above.
4. Storage: Create bucket, then paste rules above.
5. In `firebase-config.js`, fill project values from Project Settings.
