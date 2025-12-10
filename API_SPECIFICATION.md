# UCC Student Portal - API Specification

**Version**: 1.0 MVP  
**Last Updated**: December 11, 2025  
**Status**: 🟢 APPROVED FOR DEVELOPMENT

---

## 📌 Overview

This document defines all API endpoints, request/response contracts, error handling, and authentication requirements for the UCC Student Portal (Heralds). The API is built on Cloudflare Workers with a REST architecture.

---

## 🔧 API Base Information

### **Base URL**
```
Production: https://api.heralds.pages.dev
Development: https://dev-api.heralds.pages.dev
```

### **API Version**
- Current: v1
- Versioning Strategy: URL path versioning (`/api/v1/...`)
- Backward Compatibility: Maintain support for 2 previous versions

### **Authentication**
- **Method**: JWT Bearer Token
- **Header**: `Authorization: Bearer {token}`
- **Token Type**: Access Token (15 min) + Refresh Token (7 days)
- **Storage**: HttpOnly secure cookies (preferred), fallback to localStorage

### **Response Format**
All responses return JSON with consistent structure:

```json
{
  "success": true,
  "data": { },
  "error": null,
  "timestamp": "2025-12-11T10:30:00Z"
}
```

### **Error Response Format**
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "INVALID_EMAIL",
    "message": "Email format is invalid",
    "details": {}
  },
  "timestamp": "2025-12-11T10:30:00Z"
}
```

---

## 🔐 Authentication Endpoints

### **1. POST /api/v1/auth/register**
**Description**: Register a new UCC student account

**Request Body**:
```json
{
  "email": "student@ucc.edu.ph",
  "student_id": "2025001234",
  "password": "SecurePass123!",
  "first_name": "Juan",
  "last_name": "Dela Cruz"
}
```

**Validation**:
- Email must end with `@ucc.edu.ph` or `@students.ucc.edu.ph`
- Student ID format: 4-digit year + 6 digits (2025001234)
- Password: Min 8 chars, 1 uppercase, 1 number, 1 special char
- Name fields: 2-50 characters, letters only

**Response** (200 Created):
```json
{
  "success": true,
  "data": {
    "user_id": "uuid-123",
    "email": "student@ucc.edu.ph",
    "first_name": "Juan",
    "verification_sent": true,
    "verification_expires_in": 3600
  }
}
```

**Status Codes**:
- `201 Created`: Account created, verification email sent
- `400 Bad Request`: Validation failed (invalid email, weak password)
- `409 Conflict`: Email or student_id already registered
- `429 Too Many Requests`: Rate limit exceeded (5 attempts/hour)

---

### **2. POST /api/v1/auth/verify-email**
**Description**: Verify student email with token from verification link

**Query Parameters**:
- `token`: Verification token (from email link)

**Request Body** (optional if token in query):
```json
{
  "token": "verification-token-abc123"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "user_id": "uuid-123",
    "email_verified": true,
    "access_token": "jwt-token",
    "refresh_token": "refresh-jwt",
    "redirect_to": "/profile/setup"
  }
}
```

**Status Codes**:
- `200 OK`: Email verified, tokens issued
- `400 Bad Request`: Invalid or expired token
- `404 Not Found`: User not found
- `429 Too Many Requests`: Too many verification attempts

---

### **3. POST /api/v1/auth/login**
**Description**: Authenticate student with email and password

**Request Body**:
```json
{
  "email": "student@ucc.edu.ph",
  "password": "SecurePass123!"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "user_id": "uuid-123",
    "email": "student@ucc.edu.ph",
    "first_name": "Juan",
    "access_token": "jwt-token",
    "refresh_token": "refresh-jwt",
    "token_expires_in": 900,
    "user": {
      "id": "uuid-123",
      "email": "student@ucc.edu.ph",
      "profile_picture_url": "https://r2.example.com/pic.jpg",
      "bio": "UCC Student"
    }
  },
  "error": null
}
```

**Status Codes**:
- `200 OK`: Login successful
- `401 Unauthorized`: Invalid email or password
- `403 Forbidden`: Account not verified or inactive
- `429 Too Many Requests`: Rate limit (10 attempts/hour)

---

### **4. POST /api/v1/auth/logout**
**Description**: Logout and invalidate tokens

**Authentication**: Required

**Request Body**:
```json
{
  "all_devices": false
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "message": "Logged out successfully"
  }
}
```

**Status Codes**:
- `200 OK`: Logout successful
- `401 Unauthorized`: Invalid token

---

### **5. POST /api/v1/auth/refresh-token**
**Description**: Get new access token using refresh token

**Request Body**:
```json
{
  "refresh_token": "refresh-jwt-token"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "access_token": "new-jwt-token",
    "token_expires_in": 900
  }
}
```

**Status Codes**:
- `200 OK`: New token issued
- `401 Unauthorized`: Refresh token invalid or expired
- `429 Too Many Requests`: Rate limit exceeded

---

### **6. POST /api/v1/auth/forgot-password**
**Description**: Request password reset email

**Request Body**:
```json
{
  "email": "student@ucc.edu.ph"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "message": "Reset email sent if account exists",
    "reset_expires_in": 3600
  }
}
```

**Note**: Always returns success regardless of email existence (security best practice)

---

### **7. POST /api/v1/auth/reset-password**
**Description**: Reset password with reset token

**Request Body**:
```json
{
  "token": "reset-token-abc123",
  "password": "NewSecurePass123!"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "message": "Password reset successful",
    "redirect_to": "/login"
  }
}
```

**Status Codes**:
- `200 OK`: Password reset successful
- `400 Bad Request`: Invalid password format
- `401 Unauthorized`: Token invalid or expired

---

## 👤 User Endpoints

### **1. GET /api/v1/users/{user_id}**
**Description**: Get user profile

**Authentication**: Required

**Path Parameters**:
- `user_id`: UUID of the user

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "uuid-123",
    "email": "student@ucc.edu.ph",
    "student_id": "2025001234",
    "first_name": "Juan",
    "last_name": "Dela Cruz",
    "bio": "Engineering student",
    "profile_picture_url": "https://r2.example.com/pic.jpg",
    "followers_count": 42,
    "following_count": 15,
    "posts_count": 8,
    "is_following": false,
    "is_follower": false,
    "is_blocked": false,
    "created_at": "2025-01-15T10:30:00Z",
    "updated_at": "2025-12-11T14:20:00Z"
  }
}
```

**Status Codes**:
- `200 OK`: Profile retrieved
- `401 Unauthorized`: Not authenticated
- `404 Not Found`: User not found
- `403 Forbidden`: User blocked by target user

---

### **2. PUT /api/v1/users/{user_id}**
**Description**: Update user profile (own profile only)

**Authentication**: Required (user must match user_id)

**Request Body**:
```json
{
  "first_name": "Juan",
  "last_name": "Dela Cruz",
  "bio": "Updated bio here",
  "profile_picture_url": "https://r2.example.com/new-pic.jpg"
}
```

**Editable Fields**:
- `bio` (max 500 chars)
- `profile_picture_url` (from file upload)

**Non-editable Fields** (ignored if sent):
- email
- student_id
- password (use separate endpoint)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "uuid-123",
    "first_name": "Juan",
    "last_name": "Dela Cruz",
    "bio": "Updated bio here",
    "profile_picture_url": "https://r2.example.com/new-pic.jpg",
    "updated_at": "2025-12-11T14:25:00Z"
  }
}
```

**Status Codes**:
- `200 OK`: Profile updated
- `400 Bad Request`: Validation failed
- `401 Unauthorized`: Not authenticated
- `403 Forbidden`: Trying to update another user's profile

---

### **3. GET /api/v1/users/{user_id}/posts**
**Description**: Get all posts by a user

**Authentication**: Required

**Query Parameters**:
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20, max: 100)
- `sort`: `newest` or `oldest` (default: newest)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "posts": [
      {
        "id": "post-123",
        "user_id": "uuid-123",
        "content": "Just finished a great project!",
        "image_url": null,
        "reactions_count": 5,
        "comments_count": 2,
        "created_at": "2025-12-10T10:30:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 45,
      "total_pages": 3,
      "has_next": true,
      "has_prev": false
    }
  }
}
```

**Status Codes**:
- `200 OK`: Posts retrieved
- `401 Unauthorized`: Not authenticated
- `404 Not Found`: User not found

---

### **4. POST /api/v1/users/{user_id}/follow**
**Description**: Follow a user

**Authentication**: Required

**Path Parameters**:
- `user_id`: UUID of user to follow

**Request Body**: Empty or null

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "following_id": "uuid-456",
    "follower_id": "uuid-123",
    "created_at": "2025-12-11T14:30:00Z",
    "is_following": true
  }
}
```

**Status Codes**:
- `201 Created`: Now following user
- `400 Bad Request`: Cannot follow self
- `401 Unauthorized`: Not authenticated
- `404 Not Found`: User not found
- `409 Conflict`: Already following this user

---

### **5. DELETE /api/v1/users/{user_id}/follow**
**Description**: Unfollow a user

**Authentication**: Required

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "message": "Unfollowed successfully",
    "is_following": false
  }
}
```

---

## 📝 Post Endpoints

### **1. GET /api/v1/posts (Feed)**
**Description**: Get user's feed with posts from followed users

**Authentication**: Required

**Query Parameters**:
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20, max: 100)
- `sort`: `newest`, `trending`, `following` (default: newest)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "posts": [
      {
        "id": "post-123",
        "user_id": "uuid-456",
        "user": {
          "id": "uuid-456",
          "first_name": "Maria",
          "profile_picture_url": "https://r2.example.com/maria.jpg"
        },
        "content": "Just finished my thesis!",
        "image_url": "https://r2.example.com/thesis.jpg",
        "reactions_count": 12,
        "comments_count": 3,
        "my_reaction": "👍",
        "is_mine": false,
        "is_deleted": false,
        "created_at": "2025-12-10T15:30:00Z",
        "updated_at": "2025-12-10T15:30:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 156,
      "total_pages": 8,
      "has_next": true,
      "cursor": "post-123"
    }
  }
}
```

**Status Codes**:
- `200 OK`: Feed retrieved
- `401 Unauthorized`: Not authenticated

---

### **2. POST /api/v1/posts**
**Description**: Create a new post

**Authentication**: Required

**Request Body**:
```json
{
  "content": "This is my post content",
  "image_url": "https://r2.example.com/image.jpg"
}
```

**Validation**:
- `content`: 1-5000 characters, required
- `image_url`: Optional, must be valid R2 URL

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "id": "post-123",
    "user_id": "uuid-123",
    "content": "This is my post content",
    "image_url": "https://r2.example.com/image.jpg",
    "reactions_count": 0,
    "comments_count": 0,
    "created_at": "2025-12-11T14:35:00Z"
  }
}
```

**Status Codes**:
- `201 Created`: Post created
- `400 Bad Request`: Validation failed
- `401 Unauthorized`: Not authenticated
- `413 Payload Too Large`: Content exceeds limit

---

### **3. PUT /api/v1/posts/{post_id}**
**Description**: Edit a post (own posts only)

**Authentication**: Required

**Request Body**:
```json
{
  "content": "Updated post content"
}
```

**Edit Rules**:
- Only owner can edit
- Can only edit content, not image
- Timestamp shows "edited"

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "post-123",
    "content": "Updated post content",
    "is_edited": true,
    "edited_at": "2025-12-11T14:40:00Z"
  }
}
```

**Status Codes**:
- `200 OK`: Post updated
- `401 Unauthorized`: Not authenticated
- `403 Forbidden`: Not post owner
- `404 Not Found`: Post not found

---

### **4. DELETE /api/v1/posts/{post_id}**
**Description**: Delete a post (soft delete)

**Authentication**: Required

**Response** (204 No Content):
```json
{
  "success": true,
  "data": {
    "message": "Post deleted"
  }
}
```

**Status Codes**:
- `204 No Content`: Post deleted
- `401 Unauthorized`: Not authenticated
- `403 Forbidden`: Not post owner
- `404 Not Found`: Post not found

---

## 💬 Comment Endpoints

### **1. GET /api/v1/posts/{post_id}/comments**
**Description**: Get all comments on a post

**Authentication**: Required

**Query Parameters**:
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20, max: 100)
- `sort`: `newest` or `oldest` (default: newest)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "comments": [
      {
        "id": "comment-123",
        "post_id": "post-123",
        "user_id": "uuid-789",
        "user": {
          "id": "uuid-789",
          "first_name": "Carlos",
          "profile_picture_url": "https://r2.example.com/carlos.jpg"
        },
        "content": "Great post!",
        "is_mine": false,
        "is_deleted": false,
        "created_at": "2025-12-10T16:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 5,
      "total_pages": 1
    }
  }
}
```

---

### **2. POST /api/v1/posts/{post_id}/comments**
**Description**: Add a comment to a post

**Authentication**: Required

**Request Body**:
```json
{
  "content": "This is a great post!"
}
```

**Validation**:
- `content`: 1-500 characters, required

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "id": "comment-123",
    "post_id": "post-123",
    "user_id": "uuid-123",
    "content": "This is a great post!",
    "created_at": "2025-12-11T14:45:00Z"
  }
}
```

**Status Codes**:
- `201 Created`: Comment created
- `400 Bad Request`: Validation failed
- `401 Unauthorized`: Not authenticated
- `404 Not Found`: Post not found

---

### **3. DELETE /api/v1/posts/{post_id}/comments/{comment_id}**
**Description**: Delete a comment (soft delete)

**Authentication**: Required

**Response** (204 No Content):
```json
{
  "success": true,
  "data": {
    "message": "Comment deleted"
  }
}
```

---

## ⭐ Reaction Endpoints

### **1. POST /api/v1/posts/{post_id}/react**
**Description**: Add or update reaction to a post

**Authentication**: Required

**Request Body**:
```json
{
  "reaction_type": "👍"
}
```

**Allowed Reactions**: `👍`, `❤️`, `😂`, `😢`, `😡`, `🔥`

**Behavior**: 
- If user already reacted with different emoji, replace it
- If user already reacted with same emoji, no change

**Response** (201 Created / 200 OK):
```json
{
  "success": true,
  "data": {
    "id": "reaction-123",
    "post_id": "post-123",
    "user_id": "uuid-123",
    "reaction_type": "👍",
    "created_at": "2025-12-11T14:50:00Z"
  }
}
```

**Status Codes**:
- `201 Created`: New reaction added
- `200 OK`: Reaction updated
- `400 Bad Request`: Invalid reaction type
- `401 Unauthorized`: Not authenticated
- `404 Not Found`: Post not found

---

### **2. DELETE /api/v1/posts/{post_id}/react**
**Description**: Remove reaction from a post

**Authentication**: Required

**Response** (204 No Content):
```json
{
  "success": true,
  "data": {
    "message": "Reaction removed"
  }
}
```

---

### **3. GET /api/v1/posts/{post_id}/reactions**
**Description**: Get all reactions on a post

**Authentication**: Required

**Query Parameters**:
- `reaction_type`: Filter by emoji (optional)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "reactions": {
      "👍": 12,
      "❤️": 8,
      "😂": 3,
      "😢": 0,
      "😡": 0,
      "🔥": 2
    },
    "total": 25,
    "my_reaction": "👍",
    "top_reactors": [
      {
        "user_id": "uuid-456",
        "first_name": "Maria",
        "reaction_type": "❤️"
      }
    ]
  }
}
```

---

## 🔔 Notification Endpoints

### **1. GET /api/v1/notifications**
**Description**: Get user's notifications

**Authentication**: Required

**Query Parameters**:
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20)
- `unread_only`: true/false (default: false)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "notifications": [
      {
        "id": "notif-123",
        "type": "like",
        "message": "Maria reacted to your post",
        "related_user_id": "uuid-456",
        "related_user": {
          "first_name": "Maria",
          "profile_picture_url": "https://r2.example.com/maria.jpg"
        },
        "related_post_id": "post-123",
        "is_read": false,
        "created_at": "2025-12-11T14:55:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 15,
      "unread_count": 3
    }
  }
}
```

**Notification Types**:
- `like`: Someone reacted to your post
- `comment`: Someone commented on your post
- `follow`: Someone started following you
- `comment_reply`: Someone replied to your comment (future)

---

### **2. PUT /api/v1/notifications/{notification_id}/read**
**Description**: Mark notification as read

**Authentication**: Required

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "notif-123",
    "is_read": true
  }
}
```

---

### **3. DELETE /api/v1/notifications/{notification_id}**
**Description**: Delete notification

**Authentication**: Required

**Response** (204 No Content):
```json
{
  "success": true,
  "data": {
    "message": "Notification deleted"
  }
}
```

---

## 🔍 Search Endpoints

### **1. GET /api/v1/search/posts**
**Description**: Search posts by keyword

**Authentication**: Required

**Query Parameters**:
- `q`: Search query (required, min 2 chars)
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20, max: 100)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "results": [
      {
        "id": "post-123",
        "user_id": "uuid-456",
        "content": "Found content containing your query...",
        "created_at": "2025-12-10T15:30:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 45,
      "total_pages": 3
    }
  }
}
```

---

### **2. GET /api/v1/search/users**
**Description**: Search users by name

**Authentication**: Required

**Query Parameters**:
- `q`: Search query (required, min 2 chars)
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "results": [
      {
        "id": "uuid-456",
        "first_name": "Maria",
        "last_name": "Santos",
        "profile_picture_url": "https://r2.example.com/maria.jpg",
        "is_following": false
      }
    ],
    "pagination": {
      "page": 1,
      "total": 3
    }
  }
}
```

---

## 📊 Rate Limiting

**Global Rate Limits**:
- Anonymous: 30 requests/minute
- Authenticated: 300 requests/minute
- Per-user endpoint: 1000 requests/hour

**Endpoint-Specific Limits**:
- POST /auth/login: 10 attempts/hour per IP
- POST /auth/register: 5 attempts/hour per IP
- POST /posts: 100 posts/day per user
- POST /comments: 500 comments/day per user

**Rate Limit Headers**:
```
X-RateLimit-Limit: 300
X-RateLimit-Remaining: 299
X-RateLimit-Reset: 1639224600
```

---

## 🔒 Security Headers

**Required on All Responses**:
```
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

---

## 📝 Pagination Strategy

**Cursor-based Pagination** (Preferred for large datasets):
```
?cursor=post-123&limit=20&direction=next
```

**Offset Pagination** (Simpler, for MVP):
```
?page=1&limit=20&sort=newest
```

**Response Includes**:
```json
{
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 1000,
    "total_pages": 50,
    "has_next": true,
    "has_prev": false,
    "cursor": "post-123"
  }
}
```

---

## ✅ Success Checklist

- [ ] All endpoints return consistent response format
- [ ] All error responses include error code + message
- [ ] All protected endpoints check authentication
- [ ] Rate limiting implemented on sensitive endpoints
- [ ] Request validation on all inputs
- [ ] Response includes proper HTTP status codes
- [ ] API documentation updated in Storybook
- [ ] Tests written for all endpoints
- [ ] Performance testing completed
- [ ] Security audit passed (OWASP Top 10)

---

**Next Document**: DATABASE_SCHEMA.md
