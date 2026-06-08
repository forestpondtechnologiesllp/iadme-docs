# iAdMe Beta Readiness Checklist

## Purpose

This checklist must be completed before approving Public Beta and App Store / Play Store submission.

---

# Authentication

## Registration

- [ ] User registration succeeds
- [ ] Duplicate registration blocked
- [ ] Validation errors displayed correctly

## Login

- [ ] Login succeeds
- [ ] Invalid credentials rejected
- [ ] Session persists after restart

## Token Management

- [ ] Access token refresh works
- [ ] Expired token handled correctly
- [ ] Logout works

## Password Recovery

- [ ] Forgot password email sent
- [ ] Reset password works

## Email Verification

- [ ] Verification email sent
- [ ] Verification link works
- [ ] Verified state reflected in app

## Phone OTP (if enabled)

- [ ] OTP sent
- [ ] OTP verified
- [ ] Invalid OTP rejected

---

# Profile

- [ ] Update profile
- [ ] Update display name
- [ ] Update bio
- [ ] Profile loads correctly

---

# Upload Pipeline

## Upload

- [ ] Upload video
- [ ] Progress shown
- [ ] Upload completes

## Processing

- [ ] HLS generation succeeds
- [ ] Thumbnail generated
- [ ] Processing status updated

## Playback

- [ ] Video plays
- [ ] Audio works
- [ ] Full-screen works

---

# Feed

## Feed Loading

- [ ] Feed loads
- [ ] Infinite scroll works

## Location Feeds

- [ ] Nearby feed
- [ ] Local feed
- [ ] City feed
- [ ] State feed
- [ ] National feed

## Trending

- [ ] Trending feed loads
- [ ] ScrollScore updates

---

# Engagement

## Reactions

- [ ] Like
- [ ] Dislike
- [ ] Double Like

## Comments

- [ ] Add comment
- [ ] Edit comment (if supported)
- [ ] Delete comment
- [ ] Reply to comment

## Shares

- [ ] Share link generated
- [ ] Share page opens

---

# Reporting & Safety

## Report Video

- [ ] Report submitted
- [ ] Report visible in admin

## Report User

- [ ] Report submitted
- [ ] Report visible in admin

---

# Premium Content

## Unlock

- [ ] Premium preview visible
- [ ] Unlock succeeds
- [ ] Wallet debited
- [ ] Unlock persists

---

# Wallet

## Balance

- [ ] Balance loads

## Coupon Redemption

- [ ] Valid coupon works
- [ ] Duplicate redemption blocked
- [ ] Invalid coupon rejected

## Transactions

- [ ] Transaction history updates

---

# Notifications

## In-App

- [ ] Comment notification
- [ ] Reply notification
- [ ] Like notification
- [ ] Share notification

## Push Notifications

- [ ] Device registration
- [ ] Notification delivery
- [ ] Deep link routing

---

# Messaging

- [ ] Send message
- [ ] Receive message
- [ ] Conversation loads

---

# Admin Console

## Dashboard

- [ ] Dashboard metrics load

## Reports

- [ ] Report queue loads
- [ ] Resolve report
- [ ] Dismiss report

## Videos

- [ ] Delete video
- [ ] Restore video

## Users

- [ ] Deactivate user
- [ ] Reactivate user

## Comments

- [ ] Delete comment

## Coupons

- [ ] Create coupon
- [ ] Redeem coupon
- [ ] View redemption

## Operations

- [ ] Queue metrics
- [ ] Worker metrics

---

# Infrastructure

## API

- [ ] Health endpoint healthy

## Worker

- [ ] Worker healthy

## Database

- [ ] PostgreSQL healthy

## Redis

- [ ] Redis healthy

## Monitoring

- [ ] Sentry receiving events

---

# App Store Compliance

- [ ] Privacy Policy accessible
- [ ] Terms accessible
- [ ] Contact Support accessible
- [ ] Report Video available
- [ ] Report User available
- [ ] Delete Account available

---

# Production Sign-Off

## Staging Validation

- [ ] All tests passed

## Production Validation

- [ ] All tests passed

## Launch Approval

- [ ] Founder sign-off
- [ ] Production release approved

---

# Beta Exit Criteria

Beta is approved only when:

- [ ] Critical issues resolved
- [ ] No known SEV-1 issues
- [ ] No known SEV-2 issues
- [ ] Production healthy
- [ ] App Store compliance verified
- [ ] Admin console operational
- [ ] Launch approved