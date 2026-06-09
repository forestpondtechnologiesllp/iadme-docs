

# iAdMe — R5 App Store Compliance

Last Updated: June 2026

---

# Product Overview

Product Name: iAdMe

Company: FORESTPOND TECHNOLOGIES LLP

Website: https://iadme.app

Platform Type:

- Locality-first short video social platform
- User generated content
- Messaging
- Notifications
- Wallet / Stars system
- Premium content unlocks
- Advertising supported

---

# Account Deletion Compliance

Status: Complete

Implementation:

- Users can delete their account from inside the mobile application.
- Email OTP verification required before deletion.
- User videos are hidden.
- User comments are removed from public visibility.
- Followers and following relationships are removed.
- User is signed out after deletion.

Relevant Documents:

- Privacy Policy
- Terms & Conditions

---

# Privacy Policy

Status: Complete

URL:

https://iadme.app/privacy-policy

Coverage:

- Authentication
- Email
- Phone number
- Location
- Uploaded content
- Messages
- Notifications
- Wallet
- Analytics
- Advertising
- Moderation
- Account deletion
- Data retention

---

# Terms & Conditions

Status: Complete

URL:

https://iadme.app/terms-and-conditions

Coverage:

- User content ownership
- User content license
- Moderation
- Reporting
- Account suspension
- Account termination
- Wallet usage
- Premium unlocks
- Advertising
- Creator features

---

# Refund Policy

Status: Complete

URL:

https://iadme.app/refund-and-cancellation-policy

Coverage:

- Wallet transactions
- Premium unlocks
- Refund eligibility
- App Store purchases
- Play Store purchases
- Fraud prevention

---

# Deep Linking Compliance

Status: Implemented

Supported Routes:

- /v/:videoId
- /video/:videoId
- /profiles/:userId

Custom Scheme:

- iadme://video/:videoId

iOS Universal Links:

- Associated Domains configured
- apple-app-site-association created
- Team ID: KU58Q677M3

Android App Links:

- HTTPS intent filters configured
- assetlinks.json created
- Waiting for release SHA-256 fingerprint

---

# Push Notification Compliance

Status: Complete

Infrastructure:

- Firebase Cloud Messaging (FCM)
- Firebase Admin SDK

Supported Notification Types:

- Video likes
- Video comments
- Comment replies
- Shares
- Followers
- Wallet notifications
- Security notifications

Device Lifecycle:

- Device registration on login
- Token refresh handling
- Device unregistration on logout
- Device unregistration on account deletion

---

# Apple Privacy Label Mapping

Contact Information

Collected:

- Email Address
- Phone Number

Purpose:

- Authentication
- Account Management
- Security

Location

Collected:

- Approximate Location
- Potential Precise Location (permissions present)

Purpose:

- Nearby feed
- Locality ranking
- Personalization

User Content

Collected:

- Videos
- Comments
- Messages
- Profile Information

Purpose:

- Core Platform Functionality

Identifiers

Collected:

- User ID
- Device Token

Purpose:

- Authentication
- Notifications

Usage Data

Collected:

- Views
- Likes
- Comments
- Shares
- Reports
- Feed Activity

Purpose:

- Analytics
- Product Improvement

Diagnostics

Collected:

- Crash Data
- Performance Data

Provider:

- Sentry

---

# Google Play Data Safety Mapping

Collected Data

- Email
- Phone Number
- Location
- Profile Information
- Videos
- Comments
- Messages
- Device Identifiers
- Usage Data
- Crash Diagnostics

Third-Party Providers

- Firebase
- Sentry
- AWS
- Cloudflare

Data Shared With Advertisers

Current Status:

- No

Notes:

- iAdMe currently runs platform-managed advertisements.
- No third-party ad network SDK currently integrated.
- Reassess before AdMob/AppLovin/Meta integration.

Security

- Data encrypted in transit
- Account deletion available

---

# Reviewer Test Account

Status: Pending

To Be Created Before Submission.

Suggested:

reviewer@iadme.app

---

# Store Review Notes

Status: Pending

To Be Completed During Submission Preparation.

---

# Submission Checklist

R5.1 Compliance & Legal Review

- Complete

R5.2 Deep Linking Review

- Complete (pending Android SHA-256)

R5.3 Push Notification Review

- Complete

R5.4 Store Assets

- Pending

R5.5 Data Safety Forms

- Complete

R5.6 Reviewer Account & Review Notes

- Pending

R5.7 Release Candidate Validation

- Pending

Apple App Store Submission

- Pending

Google Play Submission

- Pending