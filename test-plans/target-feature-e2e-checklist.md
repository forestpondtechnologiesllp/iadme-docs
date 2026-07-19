# Target Feature E2E Checklist

## Deployment prerequisites

- Apply `20260717_create_video_target_progress.sql`.
- Apply `20260718_create_target_notification_deliveries.sql`.
- Deploy the same backend image to the API and worker services.
- Confirm the `target-notification-sweep` worker heartbeat is healthy.
- Confirm the repeatable target notification job is registered.

## Target action

- Feed, Trending, and public-profile playback show the compact Target icon.
- An inactive Target is white and outlined.
- An active Target is orange and filled.
- Tapping Target updates the count and selected state immediately.
- Returning to the same video preserves the selected state.
- The engagement sheet uses the same Target symbol and counts.

## Targets history

- Profile > History is ordered Targets, Uploads, Activity.
- Profile > History > Targets contains only targeted shorts.
- Activity does not include a Target filter.
- Target cards show targeter and achieved totals without duplicated text.
- Tapping the Target totals opens only Targeted and Achieved user lists.
- Blocked or deleted profiles do not appear in participant lists.

## Planning and completion

- A user can set, change, and remove an optional target date.
- A future date displays as the planned target date.
- A past incomplete date displays as overdue.
- Marking achieved preserves the planned date and records completion time.
- Marking incomplete removes completion time without changing the planned date.
- Completion totals update after refresh.

## Untarget and lifecycle

- Untargeting removes the short from Targets.
- Retargeting starts with no previous date or completion state.
- Changing Target to Like or Dislike clears Target progress.
- Reporting a targeted short clears Target progress.
- Deleted, private, unavailable, and blocked-author shorts cannot be targeted.
- A blocked relationship hides the short and participant identities both ways.
- Removing a temporary block or republishing restores preserved Targets.

## Notifications

- Notification Preferences contains a Targets category.
- In-app and push options can be controlled independently.
- Target milestones do not generate immediate per-action notifications.
- Consolidated count milestones are delivered once per user and video.
- Consolidated achievement milestones begin only after five targeters.
- Untargeted and blocked users do not receive later milestone updates.
- A failed sweep can retry without duplicating delivered milestones.

## Admin and website

- Admin platform metrics include Target adoption, progress, event, and delivery totals.
- The Targets dashboard loads active, unique, targeted-short, achieved, scheduled,
  overdue, completion-rate, and milestone-delivery values.
- The homepage introduces Targets without displacing existing iAdMe features.
- All eight Target FAQs expand and collapse correctly.
- Homepage Targets and FAQ layouts remain readable at desktop and mobile widths.

## Final release checks

- Backend TypeScript typecheck and production build pass.
- Flutter analysis passes.
- Backend, mobile, and website `git diff --check` pass.
- API health and worker heartbeat are healthy after deployment.
- No API or worker errors appear during the Target regression flow.
