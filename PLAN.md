# GPT Astra Social App — Product and Implementation Plan

## Product summary

Build a free, adults-only social app for friends and communities. The v1 app targets iOS and Android and uses a clean, minimal, mobile-first interface. Its four main tabs are **Home**, **Messages**, **Explore**, and **Profile**.

The project currently uses Expo Router and Expo SDK 57. Continue with Expo Router, consistent with the repository instructions and existing app structure. The selected product stack is Clerk for authentication, Convex for application data and file storage, and NativeWind for styling.

## V1 scope

### Included

- Email/password registration and sign-in with required email verification, Google OAuth, and Apple OAuth through Clerk.
- User profiles with an avatar, display name, unique handle, and bio; users can view and edit their own profile.
- Public and private accounts. Private accounts require follow approval.
- A Home feed of followed users’ posts, newest first.
- Explore with handle search and basic account suggestions.
- Posts containing one photo or one video (maximum duration 60 seconds), with an optional caption.
- Likes, comments, one-level comment replies, and deletion of a user’s own posts and comments.
- One-to-one direct messages containing text and photos.
- Push notifications for messages and follow requests, plus unread badges in the app.
- Blocking, reporting, and permanent self-serve account deletion.

### Excluded from v1

- Web app, stories, group chats, advertising, subscriptions, and an in-app activity feed.
- Push notifications for likes or comments.
- Threaded comments deeper than one reply level.
- Video attachments in direct messages.
- Personalized or ranked recommendation systems.

## User journeys and behavior

### Sign-up and returning users

1. A new user registers using email/password, Google, or Apple.
2. Email/password users verify their email before using social features.
3. After authentication, a new user completes their profile before entering Explore.
4. Returning users enter the signed-in app and can navigate the four main tabs.

### Profiles, discovery, and follows

- Handles are unique and case-insensitive. Display name, handle, avatar, and bio are editable.
- Public profiles and their posts are visible to signed-in users.
- Before a private account approves a follow request, show only its avatar, display name, handle, and bio; hide posts and follower/following lists.
- The private account owner can approve or reject follow requests. Approved followers retain access if the owner later switches the account to private. New followers must request approval.
- Explore supports handle search and simple, non-personalized suggestions. Exclude blocked users from discovery.
- Users can follow and unfollow accounts. Blocking removes both follow relationships and prevents discovery, interaction, and new messages.

### Posts and feed

- A post contains exactly one photo or one video and may have an optional caption.
- Videos are limited to 60 seconds. Posts from followed accounts appear newest first.
- Users can like posts, add comments, and reply once to a comment. Comments and replies display chronologically.
- Users can delete their own posts and comments. Post owners do not receive a separate comment-moderation permission in v1.
- Show clear loading, empty, upload-progress, error, and retry states. Uploads require a network connection.

### Direct messages

- A user can start or send in a one-to-one conversation only while both participants follow each other.
- Conversations support text and photo attachments. Group chats and video messages are excluded.
- After either participant unfollows, retain message history but pause sending until mutual following resumes.
- Blocking prevents new contact and message sending; existing history remains visible to both participants.
- Show unread message badges. Push notifications are sent for new messages.

### Notifications and safety

- Request OS notification permission after a relevant action, with context.
- Send pushes for new messages and follow requests. Do not send pushes for likes or comments.
- Users can block an account or report an account/content item.
- Persist reports for manual review by the founder using backend tooling; v1 does not include an in-app moderation console.

### Account deletion

- Provide a permanent account deletion flow.
- Delete the account’s profile, posts, comments, likes, follows/requests, messages, and stored media. Remove associated device notification registrations.
- Confirm deletion clearly before committing it, and handle cleanup so partially deleted data cannot remain accessible.

## Architecture and data model

### Service responsibilities

- **Clerk:** identity, email/password authentication, email verification, Google OAuth, and Apple OAuth.
- **Convex:** application data, authorization-checked queries and mutations, feed/discovery data, messaging data, reports, and media file storage.
- **NativeWind:** application styling, configured for the repository’s Expo/React Native setup.
- **Expo Router:** file-based routes, authentication route boundaries, and the four-tab navigation shell.
- **Expo notifications:** evaluate and configure push delivery for iOS and Android during implementation, including required credentials and notification deep links.

### Core records

Use Convex records linked to the Clerk identity identifier:

- **Users/profiles:** Clerk identity, unique normalized handle, display name, avatar reference, bio, privacy setting, timestamps, and notification token references.
- **Posts:** owner, photo/video storage reference, media type, optional caption, creation time.
- **Follows:** follower, followed user, and state (pending or approved where the target is private).
- **Likes:** user and post, with at most one like per user per post.
- **Comments:** author, post, optional parent comment for one-level replies, body, and timestamps.
- **Conversations:** the two participants and creation metadata.
- **Messages:** conversation, sender, text and/or photo reference, timestamp, and read state needed for unread badges.
- **Blocks:** blocker and blocked user.
- **Reports:** reporter, target account/content, reason, timestamp, and review status.

### Authorization and data rules

- Derive the signed-in identity from Clerk and verify it for all protected Convex operations.
- Users may edit or delete only their own profile, posts, and comments.
- Enforce private-account visibility on the backend for profile content and feeds, not only in the UI.
- Enforce approved follow state for private posts and mutual-follow state for starting/sending direct messages.
- Apply blocks to discovery, follow relationships, post interactions, and messaging.
- Prevent duplicate follows, likes, and notification registrations where applicable.
- Paginate feeds, comments, search results, and message history; avoid loading unbounded collections.

## Implementation stages

### Stage 0 — Readiness and compatibility

- Review repository instructions and current Expo SDK 57 documentation before using Expo APIs.
- Verify current official setup and compatibility documentation for Clerk, Convex, NativeWind, and Expo notifications before selecting/installing package versions.
- Create Clerk and Convex projects and configure local and production environments.
- Establish app identifiers, OAuth credentials, iOS/Android push credentials, and secure environment-variable handling.
- Confirm video byte-size, encoding, and upload limits before enabling video uploads.

### Stage 1 — App foundation

- Configure NativeWind for the current Expo SDK and React Native versions.
- Organize Expo Router route groups for signed-out, onboarding, and signed-in experiences.
- Build the four-tab shell and shared loading, error, empty, and retry patterns.
- Connect Clerk authentication state to Convex and protect signed-in routes.

### Stage 2 — Authentication and profiles

- Implement email/password registration, verification, sign-in, sign-out, Google OAuth, and Apple OAuth.
- Create the profile onboarding and profile edit/view screens.
- Enforce normalized unique handles and public/private profile visibility.

### Stage 3 — Social graph and Explore

- Implement follow/unfollow and private-account request, approve, and reject behavior.
- Implement blocking and ensure it removes both follow links and prevents new contact.
- Add handle search and basic suggestions, excluding blocked accounts and respecting privacy.

### Stage 4 — Posts and feed

- Implement photo/video selection, upload progress, retries, and post creation with optional captions.
- Build the newest-first following feed and profile post grid.
- Add likes, comments, one-level replies, and owner-only post/comment deletion.
- Enforce post visibility for private accounts in every relevant query.

### Stage 5 — Direct messaging

- Implement one-to-one conversations and message history with pagination.
- Add text and photo messages, mutual-follow authorization, read state/unread badges, and unfollow/block behavior.
- Add clear sending, upload, failure, and retry states.

### Stage 6 — Push, reports, and deletion

- Register device push tokens and deliver new-message and follow-request notifications.
- Request notification permission after a relevant action and route notification taps to the relevant screen.
- Persist reports for founder review.
- Implement permanent account deletion and associated data/media cleanup.

### Stage 7 — Beta readiness

- Review iOS and Android behavior, accessibility basics, privacy enforcement, failure states, and upload/network handling.
- Validate local and production configuration without exposing secrets in client bundles.
- Prepare beta builds and document the founder’s report-review and operational workflow.

## Acceptance criteria and verification

- Email verification is required before social use for email/password accounts; Google and Apple sign-in reach the same profile onboarding and account model.
- A user cannot claim a handle already in use, including case-only variations.
- A non-follower cannot read a private account’s posts or lists; an approved follower can.
- Switching an account to private retains approved followers and gates future requests.
- Feed ordering is newest-first and contains only visible posts from followed accounts.
- A user cannot duplicate a like, mutate another user’s post/comment, or access hidden private content through direct backend operations.
- DMs cannot be initiated or sent without mutual following; unfollow pauses sending; block prevents new contact.
- Deletion removes all account-owned content and media from user-accessible surfaces.
- Upload, network, empty, and permission-denied states give users a clear recovery path.
- Before implementation is declared complete, run `npx expo lint` and `npx tsc --noEmit` as required by repository instructions. Add or run automated tests only if requested; use the acceptance scenarios above for implementation review.

## Assumptions and open risks

### Assumptions

- Handles are case-insensitive and editable.
- After approval, followers can view a private user’s posts and follower/following lists.
- Suggestions use a simple non-personalized rule suitable for a small beta.
- Reports are reviewed manually by the founder using backend tooling; there is no moderator UI in v1.
- Uploads require a network connection; cached content may remain readable and failed actions offer retry.
- Existing chat history remains visible after unfollowing or blocking, while sending is disabled until eligibility is restored (blocking always prevents sending).
- Local development and one production environment are sufficient for the initial beta.

### Open risks to resolve during implementation

- Clerk, Convex, OAuth, push, and app release credentials have not been set up.
- Expo SDK 57 and current NativeWind/Clerk/Convex package compatibility must be verified before dependency changes. Follow the matching Expo docs and repo instructions; use `npx expo install` for Expo SDK packages. This repo has a `package-lock.json` and no `bun.lock`.
- Video maximum file size, compression/encoding policy, and storage cost limits are undecided.
- The age check for the adults-only audience is unspecified.
- Explore suggestion selection, production monitoring, detailed test coverage, and a numeric infrastructure budget are unspecified.
- Push provider behavior and operational requirements must be confirmed against current Expo docs and the chosen credential setup.
