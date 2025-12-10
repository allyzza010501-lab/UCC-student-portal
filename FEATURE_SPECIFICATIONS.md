# UCC Student Portal - Feature Specifications

**Version**: 1.0 MVP  
**Last Updated**: December 11, 2025  
**Status**: 🟢 APPROVED FOR DEVELOPMENT

---

## 📌 Overview

This document provides detailed specifications for every feature in the UCC Student Portal MVP. Each feature includes user stories, detailed workflows, edge cases, and acceptance criteria.

---

## 👤 Feature 1: User Registration & Onboarding

### **Purpose**
Create new student accounts with email verification and profile setup.

### **User Story**
```
As a UCC student,
I want to sign up for Heralds with my UCC email,
so that I can connect with other students on campus.
```

### **Registration Flow**

**Step 1: Landing Page → Sign Up Button**
- User lands on heralds.pages.dev
- Clicks "Join Heralds" button
- Redirected to `/register` page

**Step 2: Registration Form**

```
Fields:
┌─────────────────────────────┐
│ Email                       │ (required, @ucc.edu.ph)
│ Student ID                  │ (required, format 2025001234)
│ Password                    │ (required, min 8 chars, 1 upper, 1 lower, 1 number, 1 special)
│ Confirm Password            │ (required, must match)
│ First Name                  │ (required, 2-50 chars)
│ Last Name                   │ (required, 2-50 chars)
│ Terms of Service            │ (required checkbox)
│ Privacy Policy              │ (required checkbox)
└─────────────────────────────┘
```

**Step 3: Validation**

```javascript
// Real-time validation feedback
{
  email: {
    required: true,
    pattern: /^[\w.-]+@(ucc\.edu\.ph|students\.ucc\.edu\.ph)$/,
    message: "Email must be @ucc.edu.ph",
    unique: true,
    message: "This email is already registered"
  },
  student_id: {
    required: true,
    pattern: /^\d{10}$/,
    message: "Student ID must be 10 digits (e.g., 2025001234)",
    unique: true,
    message: "This student ID is already registered"
  },
  password: {
    required: true,
    minLength: 8,
    requireUppercase: true,
    requireNumber: true,
    requireSpecial: true,
    message: "Password must be at least 8 characters with uppercase, number, and special character"
  },
  passwordConfirm: {
    required: true,
    match: "password",
    message: "Passwords must match"
  },
  firstName: {
    required: true,
    minLength: 2,
    maxLength: 50,
    message: "First name must be 2-50 characters"
  },
  lastName: {
    required: true,
    minLength: 2,
    maxLength: 50,
    message: "Last name must be 2-50 characters"
  }
}
```

**Step 4: Account Creation**

- POST /api/v1/auth/register
- User created with `email_verified: false`
- Verification email sent within 3 seconds
- User redirected to `/verify-email` page

**Step 5: Email Verification**

- User clicks "Verify Email" link in inbox
- Link valid for 1 hour
- Clicking link takes user to `/verify-email?token=abc123`
- Frontend submits token to POST /api/v1/auth/verify-email
- User is logged in and redirected to `/profile/setup`

**Step 6: Profile Setup**

```
Fields:
┌─────────────────────────────┐
│ Profile Picture             │ (optional, upload image)
│ Bio                         │ (optional, max 500 chars)
│ Study Program               │ (optional, dropdown)
│ Year Level                  │ (optional, dropdown)
│ Interests (Tags)            │ (optional, multi-select)
│ Allow Notifications         │ (optional, toggle)
└─────────────────────────────┘
```

- Profile can be completed now or later
- "Skip for now" button available
- After setup, redirected to feed

### **Edge Cases**

| Case | Behavior |
|------|----------|
| Email not @ucc.edu.ph | Show error immediately, don't submit |
| Email already registered | Show "Email already in use" error |
| Verification link expired | Show "Link expired, request new one" |
| User closes email verification window | Can request email again (max 5/hour) |
| Invalid student ID format | Show format example |
| Password too weak | Show strength meter with requirements |
| Don't match ToS | Disable submit button |

### **Acceptance Criteria**

- ✅ Registration form validates all fields in real-time
- ✅ Email domain restricted to @ucc.edu.ph
- ✅ Password meets security requirements
- ✅ Verification email sent within 3 seconds
- ✅ Verification link expires after 1 hour
- ✅ User automatically logged in after email verification
- ✅ Profile setup optional but encouraged
- ✅ Success message shown after registration
- ✅ Error messages clear and actionable

---

## 🔐 Feature 2: Authentication & Login

### **Purpose**
Secure login for verified UCC students.

### **User Story**
```
As a returning student,
I want to log in with my email and password,
so that I can access my feed and connect with friends.
```

### **Login Flow**

**Step 1: Login Page**

```
Fields:
┌─────────────────────────────┐
│ Email                       │
│ Password                    │
│ Remember Me (optional)      │
│ Forgot Password?            │ (link)
└─────────────────────────────┘
Button: "Sign In"
```

**Step 2: Submission & Validation**

- POST /api/v1/auth/login
- Check email exists and is verified
- Check account not locked (after 5 failed attempts)
- Verify password matches hash
- Check account is active

**Step 3: Success**

- Access token issued (15 min)
- Refresh token issued (7 days)
- Tokens stored in HttpOnly cookies
- User redirected to feed
- Toast notification: "Welcome back, Juan!"

**Step 4: Failed Login**

```
Attempt 1-4: "Invalid email or password"
Attempt 5+: "Account locked. Try again in 15 minutes"
```

### **Edge Cases**

| Case | Behavior |
|------|----------|
| Email not in system | "Invalid email or password" (don't reveal) |
| Email not verified | "Please verify your email first" + resend button |
| Account locked (5 failed attempts) | "Account locked for 15 min" |
| Password incorrect | "Invalid email or password" |
| Account inactive/deleted | "Account not found" |

### **Acceptance Criteria**

- ✅ Login form validates email format
- ✅ Account locks after 5 failed attempts (15 min)
- ✅ Failed attempts logged for audit
- ✅ Tokens stored securely (HttpOnly cookies)
- ✅ Session created and tracked
- ✅ User agent and IP logged
- ✅ Password reset option available
- ✅ Error messages don't reveal if user exists
- ✅ Success redirects to feed

---

## 📝 Feature 3: Create & Edit Posts

### **Purpose**
Allow students to share updates, thoughts, and media with the community.

### **User Story**
```
As a student,
I want to write posts and share photos,
so that I can express myself and connect with my classmates.
```

### **Create Post Flow**

**Step 1: Post Creation Interface**

```
Location: Top of feed
┌─────────────────────────────┐
│ [Profile Pic] What's on... │ (textbox)
│                             │
│ [Image] [Emoji] [More]      │ (buttons)
└─────────────────────────────┘
```

**Step 2: Text Entry**

- Input: Textarea, max 5000 characters
- Real-time character counter (shows when > 80% full)
- Supports: Text, line breaks, mentions (@username), hashtags (#topic)
- Auto-save draft every 30 seconds (localStorage)

**Step 3: Add Image (Optional)**

- Click image button
- File picker opens
- Accepted: .jpg, .png, .gif, .webp
- Max file size: 10MB
- Image preview shown
- Ability to remove image before posting

**Step 4: Submit Post**

- POST /api/v1/posts
- Payload:
  ```json
  {
    "content": "Post text here",
    "image_url": "optional-image-url"
  }
  ```
- Image uploaded to Cloudflare R2 (if included)
- Post created with timestamp
- Feed updates immediately (optimistic update)
- Toast: "Post shared!"

### **Edit Post Flow**

**Step 1: Edit Option**

- Post owner sees "..." menu on own posts
- Click menu → "Edit"
- Edit modal opens with current content

**Step 2: Edit Content**

- Can only edit text, not image
- Same validation as create
- "Edited" timestamp shown on post

**Step 3: Submit Edit**

- PUT /api/v1/posts/{post_id}
- Feed updates with new content
- Toast: "Post updated"

### **Delete Post Flow**

**Step 1: Delete Option**

- Post owner sees "..." menu
- Click → "Delete"
- Confirmation modal: "Delete this post? This action cannot be undone."

**Step 2: Confirm Delete**

- POST /api/v1/posts/{post_id}/delete
- Post soft-deleted (marked as deleted)
- Disappears from feed immediately
- Toast: "Post deleted"

### **Edge Cases**

| Case | Behavior |
|------|----------|
| Typing same post twice | Show warning "You just posted this" |
| Very long post (3000+ chars) | Still works, warning about readability |
| Post with only spaces | Error "Post cannot be empty" |
| Image upload fails | Show error, allow retry |
| Connection lost while posting | Save draft, show "Reconnecting..." |
| Try to edit after 30+ minutes | Still allowed, shows edit timestamp |
| Click post button twice quickly | Prevent duplicate (disable button) |

### **Acceptance Criteria**

- ✅ Text posts create successfully
- ✅ Posts with images create successfully
- ✅ Character count displayed and enforced
- ✅ Post owner can edit and delete
- ✅ Edits show "edited" timestamp
- ✅ Soft deletes hide post from feed
- ✅ Draft auto-save works
- ✅ Image upload handles errors gracefully
- ✅ UI prevents duplicate submissions
- ✅ Optimistic updates (immediate UI feedback)

---

## ❤️ Feature 4: Reactions (Likes/Emoji)

### **Purpose**
Allow quick feedback on posts with emoji reactions.

### **User Story**
```
As a student,
I want to react to posts with emoji,
so that I can show appreciation without commenting.
```

### **Reaction Flow**

**Step 1: Reaction Button**

```
Reactions on post:
👍 (12)  ❤️ (8)  😂 (3)  😢 (0)  😡 (0)  🔥 (2)

My reaction: 👍 (highlighted in blue)
```

**Step 2: Add Reaction**

- Click empty emoji button
- POST /api/v1/posts/{post_id}/react
- Payload: `{ "reaction_type": "👍" }`
- Count increments immediately
- Button highlights in user's color

**Step 3: Change Reaction**

- Click different emoji while one already selected
- PUT /api/v1/posts/{post_id}/react
- Old emoji removed, new emoji added
- Counts update

**Step 4: Remove Reaction**

- Click same emoji again
- DELETE /api/v1/posts/{post_id}/react
- Count decrements
- Button no longer highlighted

### **Allowed Reactions**

```
👍 - Like/Agree
❤️ - Love
😂 - Funny
😢 - Sad
😡 - Angry
🔥 - Fire/Hot
```

### **Reaction Counter**

- Shows top 3 reactors: "Maria, Juan, and 9 others reacted"
- Click to see full list of reactors
- "You" indicator shows if user reacted

### **Edge Cases**

| Case | Behavior |
|------|----------|
| Spam clicking same reaction | Only count once (API handles idempotency) |
| Reaction while offline | Queue for when connection restored |
| React to deleted post | Show "Post no longer available" |
| Click reaction then immediately navigate | Still registers reaction |

### **Acceptance Criteria**

- ✅ Reactions add/update/remove successfully
- ✅ Counts update in real-time
- ✅ User's reaction highlighted
- ✅ Reaction list shows reactors
- ✅ Can change reaction by clicking different emoji
- ✅ Remove reaction by clicking same emoji again
- ✅ Only 6 emoji types allowed
- ✅ Counts accurate and consistent

---

## 💬 Feature 5: Comments & Discussions

### **Purpose**
Enable deeper conversations on posts.

### **User Story**
```
As a student,
I want to comment on posts and read others' comments,
so that I can have conversations and share opinions.
```

### **Comment Flow**

**Step 1: Comment Section**

```
Comments (3)
┌─────────────────────────────┐
│ [Avatar] Maria Santos       │
│ "Great post! Love this!"    │
│ Like · Reply · 2 hours ago  │
├─────────────────────────────┤
│ [Avatar] Juan Dela Cruz     │
│ "Thanks! Check this out..." │
│ Like · Reply · 1 hour ago   │
└─────────────────────────────┘

Comment Input:
[Avatar] [Type comment here...] [Send]
```

**Step 2: Add Comment**

- Click comment input field
- Type comment (max 500 chars)
- POST /api/v1/posts/{post_id}/comments
- Payload: `{ "content": "Comment text" }`
- New comment appears at top with "just now" timestamp
- Input cleared

**Step 3: View Comment Thread**

- Comments load in reverse chronological order (newest first)
- Each comment shows: Avatar, Name, Content, Timestamp, Like count
- Pagination: Load 20 comments, "Load more" button
- Scrolling to bottom auto-loads next 20

**Step 4: Comment Actions**

**Like Comment**:
- Click heart icon next to "Like"
- Like count increments
- Your like highlighted

**Delete Comment**:
- Own comments show "..." menu
- Click → "Delete"
- Confirmation modal
- Comment marked as deleted, shows "[Deleted comment]"

### **Edge Cases**

| Case | Behavior |
|------|----------|
| Post comment with only spaces | Error "Comment cannot be empty" |
| Comment twice in 2 seconds | Allow but warn "You just commented" |
| Very long comment (500 chars) | Allowed, may be hard to read |
| Try to comment on deleted post | Show "Post no longer available" |
| Comment while not authenticated | Show "Log in to comment" |
| Delete comment with replies | Comment shows "[Deleted]", replies stay |

### **Acceptance Criteria**

- ✅ Comments create successfully
- ✅ Character limit enforced (max 500)
- ✅ Comments show in correct order
- ✅ Comment count updates
- ✅ User can like comments
- ✅ User can delete own comments
- ✅ Pagination works (20 per page)
- ✅ Timestamps accurate and human-readable
- ✅ User avatars display correctly

---

## 🔔 Feature 6: Notifications

### **Purpose**
Keep students updated on interactions with their content.

### **User Story**
```
As a student,
I want to receive notifications when people engage with my posts,
so that I don't miss important interactions.
```

### **Notification Types**

| Type | Message | Action |
|------|---------|--------|
| Like | "Maria reacted to your post with ❤️" | Click → Post |
| Comment | "Juan commented on your post" | Click → Post with comment highlighted |
| Follow | "Maria started following you" | Click → Maria's profile |

### **Notification Interface**

```
Bell icon (top right): 🔔 (3)

Dropdown notification list:
┌─────────────────────────────┐
│ [✓] Maria reacted ❤️        │ 2 min ago
│ [ ] Juan commented: "Great!"│ 5 min ago
│ [ ] Alex started following  │ 1 hour ago
├─────────────────────────────┤
│        View All (12)        │
└─────────────────────────────┘
```

### **Mark as Read**

- Notification shows as unread (bold, unread circle)
- Click notification → mark as read
- PUT /api/v1/notifications/{notification_id}/read
- Unread badge count decrements
- Notification appears greyed out

### **Notification Settings**

User can disable notifications for:
- Post reactions
- Comments
- Follow requests
- Global on/off toggle

### **Edge Cases**

| Case | Behavior |
|------|----------|
| Multiple likes on same post | Aggregate: "Maria, Juan, and 3 others reacted" |
| Like, unlike, like again | Only one notification sent for latest action |
| Notification on deleted post | Notification persists but shows "Post no longer available" |
| User blocks commenter | Notification not shown |
| Comment on very old post | Notification still sent (no time limit) |

### **Acceptance Criteria**

- ✅ Notifications create when triggered
- ✅ Notification count accurate
- ✅ Mark as read works
- ✅ Unread notifications highlighted
- ✅ Click navigates to relevant content
- ✅ Settings respected (notifications toggle)
- ✅ Multiple actions aggregated intelligently
- ✅ Real-time updates (WebSocket/polling)

---

## 👥 Feature 7: User Profiles & Following

### **Purpose**
Let students showcase themselves and build connections.

### **User Story**
```
As a student,
I want to view other students' profiles and follow them,
so that I can see their posts and connect.
```

### **Profile View**

```
┌─────────────────────────────┐
│  [Background Banner]        │
│  [Avatar]  Juan Dela Cruz   │
│  @juandelacruz              │
│  "Engineering student"      │
│                             │
│  📍 Computer Science        │
│  Year: 2nd Year             │
│                             │
│  42 followers  15 following │
│                             │
│  [Message] [Follow] [...]   │
└─────────────────────────────┘
Posts by Juan
┌─────────────────────────────┐
│ [Post 1]                    │
│ [Post 2]                    │
│ [Post 3]                    │
└─────────────────────────────┘
```

### **Own Profile**

**Differences from viewing others**:
- Edit button instead of Follow
- Settings link
- Draft posts (private)
- Archive posts option

### **Edit Profile**

```
Editable Fields:
┌─────────────────────────────┐
│ Profile Picture Upload      │
│ Banner Image Upload         │
│ Bio (max 500 chars)         │
│ Study Program (dropdown)    │
│ Year Level (dropdown)       │
│ Interests (multi-select)    │
│ Private/Public Toggle       │
└─────────────────────────────┘
Button: "Save Changes"
```

### **Follow/Unfollow**

**Follow Button**:
- POST /api/v1/users/{user_id}/follow
- Button text changes to "Following"
- User appears in your following list
- Their posts appear in your feed

**Unfollow Button**:
- DELETE /api/v1/users/{user_id}/follow
- Button changes back to "Follow"
- User's posts removed from feed
- Notification sent to user (optional)

### **Block User**

**Block Option**:
- Click "..." menu on user's profile
- "Block user" option
- User can no longer see your posts
- You can't see their posts/profile
- Both get notification (optional)

### **Edge Cases**

| Case | Behavior |
|------|----------|
| Try to follow self | Button disabled, tooltip "Can't follow yourself" |
| Follow someone who blocked you | "This user has blocked you" error |
| User deletes profile while viewing | Show "Profile no longer available" |
| Follow someone mid-scroll | Feed updates with new posts from user |
| Unblock user | Mutual visibility restored |

### **Acceptance Criteria**

- ✅ Profile displays all information correctly
- ✅ Avatar and banner images load
- ✅ Edit profile saves successfully
- ✅ Follow/unfollow works
- ✅ Following count accurate
- ✅ Block/unblock prevents visibility
- ✅ Profile can be set to private
- ✅ User's own posts appear on profile
- ✅ Stats (followers, following, posts) update correctly

---

## 🔍 Feature 8: Search

### **Purpose**
Help students find posts and other students.

### **User Story**
```
As a student,
I want to search for topics and people,
so that I can find specific discussions and classmates.
```

### **Search Interface**

```
┌──────────────────────────────────────┐
│ 🔍 [Search posts and people...]      │
└──────────────────────────────────────┘
```

### **Search Posts**

- Query: Min 2 characters
- Search matches: Post content, hashtags
- Results show: Post preview, author, timestamp, engagement count
- Click result → full post
- GET /api/v1/search/posts?q=python&page=1

### **Search People**

- Query: Min 2 characters
- Search matches: First name, last name, username
- Results show: Avatar, name, bio, follow button
- Click result → profile
- GET /api/v1/search/users?q=maria&page=1

### **Search Results**

```
Tabs: [All] [Posts] [People]

Posts Tab:
┌─────────────────────────────┐
│ Post result 1               │
├─────────────────────────────┤
│ Post result 2               │
├─────────────────────────────┤
│ Post result 3               │
└─────────────────────────────┘
[Load More]

People Tab:
┌─────────────────────────────┐
│ [Avatar] Maria Santos  [F]  │
├─────────────────────────────┤
│ [Avatar] Juan Dela Cruz [F] │
└─────────────────────────────┘
```

### **Search Features**

- **Real-time suggestions** as user types
- **Trending topics** if search empty
- **Recent searches** stored (last 5)
- **Clear history** option
- **Filters**: By date, engagement, people

### **Edge Cases**

| Case | Behavior |
|------|----------|
| No results | "No posts found. Try different keywords" |
| Search for blocked user | User doesn't appear in results |
| Search for deleted post | Doesn't appear in results |
| Very long search query | Truncate to 500 chars |
| Search with special characters | Sanitize before searching |

### **Acceptance Criteria**

- ✅ Search returns relevant results
- ✅ Min 2 character requirement enforced
- ✅ Results paginated (20 per page)
- ✅ Suggestions work in real-time
- ✅ Trending topics displayed when empty
- ✅ Recent searches stored and displayed
- ✅ Results exclude deleted content
- ✅ Results exclude blocked users

---

## ⚙️ Cross-Feature Requirements

### **Performance**

| Feature | Target | Actual |
|---------|--------|--------|
| Feed load | < 2 sec | TBD |
| Post creation | < 1 sec | TBD |
| Search results | < 1 sec | TBD |
| Profile load | < 1.5 sec | TBD |

### **Accessibility**

- ✅ All features keyboard navigable
- ✅ Screen reader compatible
- ✅ Color contrast meets WCAG AA
- ✅ Touch targets minimum 44x44px

### **Data Validation**

- ✅ All inputs validated client-side
- ✅ All inputs validated server-side
- ✅ XSS protection (sanitize HTML)
- ✅ SQL injection prevention (parameterized queries)

### **Error Handling**

- ✅ Network errors handled gracefully
- ✅ Server errors show user-friendly messages
- ✅ Validation errors highlighted inline
- ✅ Retry logic for failed requests

---

## ✅ MVP Feature Checklist

- [ ] User Registration & Onboarding
- [ ] Authentication & Login
- [ ] Create & Edit Posts
- [ ] Reactions (6 emoji types)
- [ ] Comments & Discussions
- [ ] Notifications
- [ ] User Profiles & Following
- [ ] Search (Posts & People)
- [ ] All features responsive (mobile + desktop)
- [ ] Performance targets met
- [ ] Security requirements met
- [ ] Accessibility standards met

---

**Next Document**: SECURITY_CHECKLIST.md
