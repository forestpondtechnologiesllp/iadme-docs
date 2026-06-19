

# iAdMe — Apple App Review Notes

Last Updated: June 2026

---

# App Overview

iAdMe is a locality-first short video social platform.

Users can:

- Discover nearby and local short videos
- Upload videos
- Like, comment, share, and report content
- Follow creators
- Send direct messages
- Receive notifications
- Unlock premium videos using Stars

---

# Reviewer Test Account

Reviewer Email:

iadmeappuser@gmail.com

Reviewer Password:

iAdMeappuser

Notes:

- Reviewer account is a normal user account.
- Reviewer account is not an administrator account.
- Reviewer account has access to application features required for review.

---

# Location Permission Requirement

Important:

iAdMe is a locality-first content discovery platform.

Location access is required to:

- Determine nearby content
- Determine locality feed
- Determine city feed
- Determine state feed
- Determine country feed
- Support feed ranking and discovery

Location permission is strongly recommended because iAdMe is a locality-first content platform.

If location access is unavailable, the application continues to function using broader geographic feed discovery.

For review, please allow location permission when prompted.

---

# Authentication Flow

Email verification is mandatory.

Review flow:

1. Login using the reviewer account.
2. Email verification is required.
3. Phone verification is optional.
4. After authentication, the reviewer can access all application features.

---

# Content Availability

The reviewer account can immediately access available content after login and location permission approval.

The reviewer can:

- View videos
- Open creator profiles
- View comments
- Create comments
- Like videos
- Share videos
- Report videos

---

# Upload Testing

The reviewer account can upload videos.

Reviewers may test:

- Video upload
- Video processing
- Video playback
- Video visibility

---

# Premium Content Testing

The reviewer account is provided with starter Stars.

Reviewers can:

- Unlock premium videos
- Test premium video playback
- Test premium content access flows

Current allocation:

- 100 Stars available
- Sufficient balance to unlock multiple premium videos

---

# Messaging

The application supports direct messaging.

Current implementation supports:

- Text messages
- Conversation threads
- Notifications

Media attachments are not currently enabled.

---

# Notifications

The application uses Firebase Cloud Messaging.

Reviewers may receive:

- Like notifications
- Comment notifications
- Reply notifications
- Follow notifications
- Wallet notifications

Notification taps deep-link into the appropriate screen.

---

# Account Deletion

Account deletion is available directly inside the application.

Path:

Profile → Delete Account

Account deletion requires email OTP verification.

---

# Deep Links

Supported deep links:

- https://iadme.app/v/{videoId}
- https://iadme.app/video/{videoId}
- https://iadme.app/profiles/{userId}

Universal Links and Android App Links are implemented.

---

# Contact

Company:

FORESTPOND TECHNOLOGIES LLP

Website:

https://iadme.app

Support:

support@iadme.app
