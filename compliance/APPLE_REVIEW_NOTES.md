# iAdMe — Apple App Review Notes

**Last Updated:** June 2026  
**Submission:** Version 1.0 (Build 11)

---

# App Overview

**iAdMe** is a locality-first short video social platform.

Features:

- Short video feed
- Uploads
- Likes, comments, shares and reports
- Follows
- Direct messages
- Notifications
- Premium video unlocks using Stars

---

# Reviewer Test Accounts

## Reviewer Account A

- **Email:** iadme.review@gmail.com
- **Password:** iadme.review@gmail.com
- **Display Name:** USER A

## Reviewer Account B

- **Email:** iadmeappuser@gmail.com
- **Password:** iadmeappuser@gmail.com
- **Display Name:** USER B

### Notes

- Both reviewer accounts are standard user accounts.
- Neither account has administrator privileges.
- Both accounts have full access to all application features required for review.
- Both accounts contain pre-populated content and interactions for testing.

---

# Location Permission

iAdMe is a locality-first content discovery platform.

Location permission improves nearby, locality, city, state and country feed discovery.

If denied, the app continues using broader geographic content.

---

# Authentication & Registration

Email verification is mandatory.

### New User Registration

New users are required to:

- Verify their email address
- Accept the **Terms & Conditions (EULA)**
- Accept the **Privacy Policy**

### Existing Reviewer Accounts

Returning users may sign in directly using either reviewer account provided above.

---

# Content Availability

The reviewer accounts contain pre-populated content for testing:

- Feed
- Trending
- Public & premium videos
- Creator profiles
- Comments
- Reactions
- Direct messages
- Reporting
- Blocking

---

# Upload Testing

Reviewer accounts can upload videos.

Reviewers may test:

- Video upload
- Video processing
- Video playback
- Video visibility

---

# Premium Content Testing

Reviewer accounts include sufficient Stars to test premium video unlocking and playback.

---

# Messaging

The application supports direct messaging.

Supports text messaging, conversation threads and notifications.

Users can block abusive users directly from the conversation menu.

---

# Notifications

The application uses Firebase Cloud Messaging (FCM).

Supports like, comment, reply, follow and wallet notifications with deep links.

---

# User Safety & Moderation

The application includes multiple moderation and user safety mechanisms.

Users can:

- Report videos
- Report comments
- Report user accounts
- Block abusive users

When a user is blocked:

- The blocked user's profile becomes inaccessible.
- The blocked user's videos are removed from the blocking user's Feed and Trending.
- The blocked user's comments and reactions are hidden.
- Direct messaging between the two users is disabled until the user is unblocked.

Users are also required to accept the **Terms & Conditions (EULA)** and **Privacy Policy** during account registration.

The Terms & Conditions include a **zero-tolerance policy** for objectionable content and abusive users.

---

# Screen Recording

A physical iOS device screen recording has been attached demonstrating:

- User registration
- Acceptance of the Terms & Conditions
- Acceptance of the Privacy Policy
- Reporting objectionable content
- Reporting abusive users
- Blocking abusive users

---

# Account Deletion

Account deletion is available directly within the application.

**Navigation Path**

**Profile → Delete Account**

Account deletion requires email OTP verification.

---

# Deep Links

Supported deep links:

- `https://iadme.app/v/{videoId}`
- `https://iadme.app/video/{videoId}`
- `https://iadme.app/profiles/{userId}`

Universal Links and Android App Links are implemented.

---

# Contact

**Company:** FORESTPOND TECHNOLOGIES LLP

**Website:** https://iadme.app

**Support:** support@iadme.app