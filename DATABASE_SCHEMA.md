# UCC Student Portal - Database Schema

**Version**: 1.0 MVP  
**Last Updated**: December 11, 2025  
**Database**: Cloudflare D1 (SQLite)  
**Status**: 🟢 APPROVED FOR DEVELOPMENT

---

## 📌 Overview

This document defines the complete database schema for the UCC Student Portal. The database is hosted on Cloudflare D1 (SQLite-compatible) with 8 core tables and 4 relationship tables supporting 13,000+ students.

---

## 🏗️ Database Architecture

### **Design Principles**

1. **Normalization**: 3NF (Third Normal Form) to reduce redundancy
2. **Scalability**: Designed to support 13,000+ active users
3. **Performance**: Indexes on frequently queried fields
4. **Soft Deletes**: Records marked as deleted, not removed (audit trail)
5. **Timestamps**: `created_at` and `updated_at` on all tables
6. **UUIDs**: Primary keys are UUIDs for distributed systems

### **Connection Pooling**

```javascript
// Cloudflare D1 Connection
const db = env.DB; // Provided by Cloudflare
// Max connections: 20 (D1 limit)
// Query timeout: 30 seconds
```

---

## 📋 Core Tables

### **1. Users Table**

**Purpose**: Store student account information

```sql
CREATE TABLE users (
  id TEXT PRIMARY KEY,                    -- UUID
  email TEXT UNIQUE NOT NULL,              -- student@ucc.edu.ph
  student_id TEXT UNIQUE NOT NULL,         -- 2025001234
  password_hash TEXT NOT NULL,             -- Bcrypt hash (min 12 rounds)
  first_name TEXT NOT NULL,                -- 2-50 chars
  last_name TEXT NOT NULL,                 -- 2-50 chars
  bio TEXT,                                -- 0-500 chars, optional
  profile_picture_url TEXT,                -- R2 URL, optional
  email_verified BOOLEAN DEFAULT FALSE,    -- Email verification status
  email_verified_at TIMESTAMP,             -- When email was verified
  verification_token TEXT,                 -- For email verification
  verification_expires_at TIMESTAMP,       -- Token expiration
  reset_token TEXT,                        -- For password reset
  reset_expires_at TIMESTAMP,              -- Token expiration
  is_active BOOLEAN DEFAULT TRUE,          -- Account status
  last_login_at TIMESTAMP,                 -- Last successful login
  login_attempts INT DEFAULT 0,            -- Failed login counter
  login_locked_until TIMESTAMP,            -- Account lockout time
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP                     -- Soft delete marker
);

-- Indexes for performance
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_student_id ON users(student_id);
CREATE INDEX idx_users_email_verified ON users(email_verified);
CREATE INDEX idx_users_is_active ON users(is_active);
```

**Constraints**:
- Email must match `@ucc.edu.ph` or `@students.ucc.edu.ph`
- Student ID format: 4-digit year + 6 digits (2025001234)
- Password stored as Bcrypt hash (never plain text)
- Email and student_id are globally unique

**Example Data**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "juan.delacruz@ucc.edu.ph",
  "student_id": "2025001234",
  "first_name": "Juan",
  "last_name": "Dela Cruz",
  "bio": "Engineering student, love coding",
  "email_verified": true,
  "is_active": true
}
```

---

### **2. Posts Table**

**Purpose**: Store user-created posts

```sql
CREATE TABLE posts (
  id TEXT PRIMARY KEY,                     -- UUID
  user_id TEXT NOT NULL,                   -- Foreign key to users
  content TEXT NOT NULL,                   -- 1-5000 chars
  image_url TEXT,                          -- R2 URL, optional
  is_edited BOOLEAN DEFAULT FALSE,
  edited_at TIMESTAMP,                     -- When last edited
  reactions_count INT DEFAULT 0,           -- Denormalized count
  comments_count INT DEFAULT 0,            -- Denormalized count
  is_deleted BOOLEAN DEFAULT FALSE,        -- Soft delete
  deleted_at TIMESTAMP,                    -- When deleted
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Indexes
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);
CREATE INDEX idx_posts_is_deleted ON posts(is_deleted);
CREATE INDEX idx_posts_reactions_count ON posts(reactions_count DESC);
```

**Constraints**:
- user_id must exist in users table
- content required and non-empty
- Timestamps automatically managed

**Example Data**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440001",
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "content": "Just finished my thesis! Feeling accomplished 🎓",
  "image_url": "https://r2.example.com/thesis-photo.jpg",
  "reactions_count": 12,
  "comments_count": 3,
  "created_at": "2025-12-10T15:30:00Z"
}
```

---

### **3. Comments Table**

**Purpose**: Store comments on posts

```sql
CREATE TABLE comments (
  id TEXT PRIMARY KEY,                     -- UUID
  post_id TEXT NOT NULL,                   -- Foreign key to posts
  user_id TEXT NOT NULL,                   -- Who commented
  content TEXT NOT NULL,                   -- 1-500 chars
  is_deleted BOOLEAN DEFAULT FALSE,        -- Soft delete
  deleted_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (post_id) REFERENCES posts(id),
  FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Indexes
CREATE INDEX idx_comments_post_id ON comments(post_id);
CREATE INDEX idx_comments_user_id ON comments(user_id);
CREATE INDEX idx_comments_created_at ON comments(created_at DESC);
```

**Constraints**:
- post_id and user_id must exist in respective tables
- content required, max 500 chars
- One comment per user per post allowed (no duplicates)

**Example Data**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440002",
  "post_id": "550e8400-e29b-41d4-a716-446655440001",
  "user_id": "550e8400-e29b-41d4-a716-446655440003",
  "content": "Congratulations! This is awesome! 🎉",
  "created_at": "2025-12-10T16:00:00Z"
}
```

---

### **4. Reactions Table**

**Purpose**: Store emoji reactions to posts (likes, loves, etc.)

```sql
CREATE TABLE reactions (
  id TEXT PRIMARY KEY,                     -- UUID
  post_id TEXT NOT NULL,                   -- Foreign key to posts
  user_id TEXT NOT NULL,                   -- Who reacted
  reaction_type TEXT NOT NULL,             -- 👍, ❤️, 😂, 😢, 😡, 🔥
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (post_id) REFERENCES posts(id),
  FOREIGN KEY (user_id) REFERENCES users(id),
  UNIQUE(post_id, user_id)                 -- One reaction per user per post
);

-- Indexes
CREATE INDEX idx_reactions_post_id ON reactions(post_id);
CREATE INDEX idx_reactions_user_id ON reactions(user_id);
CREATE INDEX idx_reactions_reaction_type ON reactions(reaction_type);
```

**Constraints**:
- One reaction per user per post (updated, not duplicated)
- reaction_type must be one of: 👍, ❤️, 😂, 😢, 😡, 🔥

**Example Data**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440004",
  "post_id": "550e8400-e29b-41d4-a716-446655440001",
  "user_id": "550e8400-e29b-41d4-a716-446655440003",
  "reaction_type": "❤️",
  "created_at": "2025-12-10T16:05:00Z"
}
```

---

### **5. Follows Table**

**Purpose**: Store follower relationships between users

```sql
CREATE TABLE follows (
  id TEXT PRIMARY KEY,                     -- UUID
  follower_id TEXT NOT NULL,               -- User doing the following
  following_id TEXT NOT NULL,              -- User being followed
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (follower_id) REFERENCES users(id),
  FOREIGN KEY (following_id) REFERENCES users(id),
  UNIQUE(follower_id, following_id),       -- Prevent duplicate follows
  CHECK (follower_id != following_id)      -- Can't follow self
);

-- Indexes
CREATE INDEX idx_follows_follower_id ON follows(follower_id);
CREATE INDEX idx_follows_following_id ON follows(following_id);
```

**Constraints**:
- follower_id and following_id must be different
- Prevent duplicate follows (user can only follow another once)
- Both IDs must exist in users table

**Example Data**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440005",
  "follower_id": "550e8400-e29b-41d4-a716-446655440000",
  "following_id": "550e8400-e29b-41d4-a716-446655440003",
  "created_at": "2025-12-10T10:00:00Z"
}
```

---

### **6. Notifications Table**

**Purpose**: Store user notifications for real-time updates

```sql
CREATE TABLE notifications (
  id TEXT PRIMARY KEY,                     -- UUID
  user_id TEXT NOT NULL,                   -- Recipient
  type TEXT NOT NULL,                      -- like, comment, follow, comment_reply
  message TEXT NOT NULL,                   -- Human-readable message
  related_user_id TEXT,                    -- User who triggered (Foreign key)
  related_post_id TEXT,                    -- Post involved (Foreign key)
  related_comment_id TEXT,                 -- Comment involved (Foreign key)
  is_read BOOLEAN DEFAULT FALSE,
  read_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (related_user_id) REFERENCES users(id),
  FOREIGN KEY (related_post_id) REFERENCES posts(id),
  FOREIGN KEY (related_comment_id) REFERENCES comments(id)
);

-- Indexes
CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_is_read ON notifications(is_read);
CREATE INDEX idx_notifications_created_at ON notifications(created_at DESC);
CREATE INDEX idx_notifications_user_read ON notifications(user_id, is_read);
```

**Notification Types**:
- `like`: Someone reacted to your post
- `comment`: Someone commented on your post
- `follow`: Someone started following you
- `comment_reply`: Someone replied to your comment (future)

**Example Data**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440006",
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "type": "like",
  "message": "Maria reacted to your post with ❤️",
  "related_user_id": "550e8400-e29b-41d4-a716-446655440003",
  "related_post_id": "550e8400-e29b-41d4-a716-446655440001",
  "is_read": false,
  "created_at": "2025-12-10T16:10:00Z"
}
```

---

### **7. Blocks Table**

**Purpose**: Allow users to block other users

```sql
CREATE TABLE blocks (
  id TEXT PRIMARY KEY,                     -- UUID
  blocker_id TEXT NOT NULL,                -- User doing the blocking
  blocked_id TEXT NOT NULL,                -- User being blocked
  reason TEXT,                             -- Optional reason
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (blocker_id) REFERENCES users(id),
  FOREIGN KEY (blocked_id) REFERENCES users(id),
  UNIQUE(blocker_id, blocked_id),          -- Can only block once
  CHECK (blocker_id != blocked_id)         -- Can't block self
);

-- Indexes
CREATE INDEX idx_blocks_blocker_id ON blocks(blocker_id);
CREATE INDEX idx_blocks_blocked_id ON blocks(blocked_id);
```

**Constraints**:
- blocker_id and blocked_id must be different
- One block per user pair

**Example Data**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440007",
  "blocker_id": "550e8400-e29b-41d4-a716-446655440000",
  "blocked_id": "550e8400-e29b-41d4-a716-446655440008",
  "reason": "Inappropriate behavior",
  "created_at": "2025-12-10T12:00:00Z"
}
```

---

### **8. Reports Table**

**Purpose**: Track user reports for moderation

```sql
CREATE TABLE reports (
  id TEXT PRIMARY KEY,                     -- UUID
  reporter_id TEXT NOT NULL,               -- Who reported
  reported_item_type TEXT NOT NULL,        -- post, comment, user
  reported_item_id TEXT NOT NULL,          -- UUID of reported item
  reason TEXT NOT NULL,                    -- Report reason
  description TEXT,                        -- Detailed description
  status TEXT DEFAULT 'pending',           -- pending, investigating, resolved, dismissed
  reviewed_by TEXT,                        -- Admin who reviewed (Foreign key)
  review_notes TEXT,
  action_taken TEXT,                       -- warn, suspend, delete, none
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  reviewed_at TIMESTAMP,
  FOREIGN KEY (reporter_id) REFERENCES users(id),
  FOREIGN KEY (reviewed_by) REFERENCES users(id)
);

-- Indexes
CREATE INDEX idx_reports_status ON reports(status);
CREATE INDEX idx_reports_created_at ON reports(created_at DESC);
CREATE INDEX idx_reports_reported_item ON reports(reported_item_type, reported_item_id);
```

**Report Reasons**:
- Inappropriate content
- Harassment or bullying
- Spam or advertising
- Copyright violation
- Misinformation
- Other

---

## 🔄 Migration Scripts

### **Initial Setup (v1.0.0)**

```sql
-- Run in order

-- 1. Create users table
CREATE TABLE users (
  id TEXT PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  student_id TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  first_name TEXT NOT NULL,
  last_name TEXT NOT NULL,
  bio TEXT,
  profile_picture_url TEXT,
  email_verified BOOLEAN DEFAULT FALSE,
  email_verified_at TIMESTAMP,
  verification_token TEXT,
  verification_expires_at TIMESTAMP,
  reset_token TEXT,
  reset_expires_at TIMESTAMP,
  is_active BOOLEAN DEFAULT TRUE,
  last_login_at TIMESTAMP,
  login_attempts INT DEFAULT 0,
  login_locked_until TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP
);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_student_id ON users(student_id);
CREATE INDEX idx_users_email_verified ON users(email_verified);
CREATE INDEX idx_users_is_active ON users(is_active);

-- 2. Create posts table
CREATE TABLE posts (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  content TEXT NOT NULL,
  image_url TEXT,
  is_edited BOOLEAN DEFAULT FALSE,
  edited_at TIMESTAMP,
  reactions_count INT DEFAULT 0,
  comments_count INT DEFAULT 0,
  is_deleted BOOLEAN DEFAULT FALSE,
  deleted_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);
CREATE INDEX idx_posts_is_deleted ON posts(is_deleted);

-- 3. Create comments table
CREATE TABLE comments (
  id TEXT PRIMARY KEY,
  post_id TEXT NOT NULL,
  user_id TEXT NOT NULL,
  content TEXT NOT NULL,
  is_deleted BOOLEAN DEFAULT FALSE,
  deleted_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (post_id) REFERENCES posts(id),
  FOREIGN KEY (user_id) REFERENCES users(id)
);
CREATE INDEX idx_comments_post_id ON comments(post_id);
CREATE INDEX idx_comments_user_id ON comments(user_id);

-- 4. Create reactions table
CREATE TABLE reactions (
  id TEXT PRIMARY KEY,
  post_id TEXT NOT NULL,
  user_id TEXT NOT NULL,
  reaction_type TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (post_id) REFERENCES posts(id),
  FOREIGN KEY (user_id) REFERENCES users(id),
  UNIQUE(post_id, user_id)
);
CREATE INDEX idx_reactions_post_id ON reactions(post_id);
CREATE INDEX idx_reactions_user_id ON reactions(user_id);

-- 5. Create follows table
CREATE TABLE follows (
  id TEXT PRIMARY KEY,
  follower_id TEXT NOT NULL,
  following_id TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (follower_id) REFERENCES users(id),
  FOREIGN KEY (following_id) REFERENCES users(id),
  UNIQUE(follower_id, following_id),
  CHECK (follower_id != following_id)
);
CREATE INDEX idx_follows_follower_id ON follows(follower_id);
CREATE INDEX idx_follows_following_id ON follows(following_id);

-- 6. Create notifications table
CREATE TABLE notifications (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  type TEXT NOT NULL,
  message TEXT NOT NULL,
  related_user_id TEXT,
  related_post_id TEXT,
  related_comment_id TEXT,
  is_read BOOLEAN DEFAULT FALSE,
  read_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (related_user_id) REFERENCES users(id),
  FOREIGN KEY (related_post_id) REFERENCES posts(id),
  FOREIGN KEY (related_comment_id) REFERENCES comments(id)
);
CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_is_read ON notifications(is_read);

-- 7. Create blocks table
CREATE TABLE blocks (
  id TEXT PRIMARY KEY,
  blocker_id TEXT NOT NULL,
  blocked_id TEXT NOT NULL,
  reason TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (blocker_id) REFERENCES users(id),
  FOREIGN KEY (blocked_id) REFERENCES users(id),
  UNIQUE(blocker_id, blocked_id),
  CHECK (blocker_id != blocked_id)
);
CREATE INDEX idx_blocks_blocker_id ON blocks(blocker_id);
CREATE INDEX idx_blocks_blocked_id ON blocks(blocked_id);

-- 8. Create reports table
CREATE TABLE reports (
  id TEXT PRIMARY KEY,
  reporter_id TEXT NOT NULL,
  reported_item_type TEXT NOT NULL,
  reported_item_id TEXT NOT NULL,
  reason TEXT NOT NULL,
  description TEXT,
  status TEXT DEFAULT 'pending',
  reviewed_by TEXT,
  review_notes TEXT,
  action_taken TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  reviewed_at TIMESTAMP,
  FOREIGN KEY (reporter_id) REFERENCES users(id),
  FOREIGN KEY (reviewed_by) REFERENCES users(id)
);
CREATE INDEX idx_reports_status ON reports(status);
CREATE INDEX idx_reports_created_at ON reports(created_at DESC);
```

---

## 📊 Entity Relationship Diagram (ASCII)

```
┌──────────────┐
│    users     │
├──────────────┤
│ id (PK)      │
│ email        │
│ student_id   │
│ password_hash│
│ first_name   │
│ last_name    │
│ bio          │
│ created_at   │
└──────────────┘
       │
       ├─ 1:N ─→ posts
       │
       ├─ 1:N ─→ comments
       │
       ├─ 1:N ─→ reactions
       │
       ├─ 1:N ─→ follows (follower)
       │
       ├─ 1:N ─→ follows (following)
       │
       ├─ 1:N ─→ notifications
       │
       ├─ 1:N ─→ blocks (blocker)
       │
       └─ 1:N ─→ blocks (blocked)

┌──────────────┐         ┌───────────────┐
│    posts     │────────→│   comments    │
├──────────────┤1       N├───────────────┤
│ id (PK)      │         │ id (PK)       │
│ user_id (FK) │         │ post_id (FK)  │
│ content      │         │ user_id (FK)  │
│ image_url    │         │ content       │
│ reactions_cnt│         │ created_at    │
│ comments_cnt │         └───────────────┘
│ created_at   │
└──────────────┘
       │
       1:N
       │
    ┌──────────────┐
    │  reactions   │
    ├──────────────┤
    │ id (PK)      │
    │ post_id (FK) │
    │ user_id (FK) │
    │ reaction_type│
    │ created_at   │
    └──────────────┘
```

---

## 🚀 Performance Optimization

### **Query Optimization Strategy**

1. **Denormalization for Speed**:
   - `posts.reactions_count` and `posts.comments_count` updated in real-time
   - Avoids COUNT(*) queries on every feed render

2. **Indexes on Frequent Queries**:
   - User login: `idx_users_email`, `idx_users_is_active`
   - Feed generation: `idx_posts_created_at DESC`, `idx_posts_is_deleted`
   - User profiles: `idx_follows_follower_id`, `idx_follows_following_id`
   - Notifications: `idx_notifications_user_id`, `idx_notifications_is_read`

3. **Pagination Strategy**:
   - Use LIMIT + OFFSET for MVP
   - Cursor-based pagination for production (future)

### **Expected Query Performance**

```
Operation                          Expected Time
─────────────────────────────────────────────────
Get user profile                   < 10ms
Get user feed (20 posts)           < 50ms
Search posts (100 results)         < 100ms
Get post comments (20 comments)    < 30ms
Create post                        < 20ms
Create reaction                    < 15ms
Get notifications (20)             < 40ms
Get follower list (1000 users)     < 100ms
```

---

## 🔒 Data Privacy & Security

### **PII Protection**

- Password: Bcrypt hash (never stored plain)
- Email: Encrypted at rest in D1
- Student ID: Indexed but not exposed in API responses
- Tokens: Rotate every 15 minutes (access) and 7 days (refresh)

### **Soft Delete Strategy**

- `deleted_at` timestamp records when item was deleted
- Soft deletes preserve referential integrity
- Hard deletes only after 90 days (compliance window)
- Soft deleted content excluded from feeds automatically

---

## ✅ Success Checklist

- [ ] All tables created successfully
- [ ] All indexes created for performance
- [ ] Foreign key constraints enforced
- [ ] Unique constraints prevent duplicates
- [ ] Soft delete markers on content tables
- [ ] Timestamps auto-managed (created_at, updated_at)
- [ ] Database initialized with test data
- [ ] Performance tested with 13,000+ user dataset
- [ ] Backup strategy implemented (Cloudflare auto-backup)
- [ ] Security audit passed

---

**Next Document**: AUTHENTICATION_GUIDE.md
