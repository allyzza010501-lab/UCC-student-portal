# 🚀 Monday Kickoff Guide - Team Setup Instructions

**Date**: Monday, December 16, 2025  
**Time**: 9:00 AM  
**Duration**: 30 minutes standup + full day of setup  
**Audience**: All developers, designers, and team members

---

## ✅ Pre-Kickoff Checklist (Complete by Sunday)

Before Monday, every team member needs:

| Item | What It Is | Why | Check |
|------|-----------|-----|-------|
| **GitHub Account** | Free account at github.com | To access code | [ ] |
| **Node.js 18+** | From nodejs.org | To run Vite, npm commands | [ ] |
| **Git** | From git-scm.com | To clone repository | [ ] |
| **A Code Editor** | VS Code (recommended) | To write code | [ ] |
| **Cloudflare Account** | Free account at cloudflare.com | For backend deployment | [ ] |
| **This guide printed** | These instructions | To follow along | [ ] |

**Haven't done this?** DO IT NOW BEFORE MONDAY!

---

## 📱 Monday Morning Schedule

```
9:00 AM - 9:10 AM   → STANDUP (What is Heralds?)
9:10 AM - 9:20 AM   → TECH OVERVIEW (How we're building it)
9:20 AM - 9:30 AM   → YOUR FIRST TASKS (What you do now)
9:30 AM onwards      → EXECUTION (Get coding!)
```

---

## 📚 Documents You'll Need Monday

**Print or bookmark these**:

1. **README.md** ← START HERE (15 mins)
   - Project overview
   - How to run locally
   - Directory structure

2. **Your role-specific guide**:
   - **Frontend dev**: FRONTEND_SETUP.md
   - **Backend dev**: BACKEND_SETUP.md
   - **Designer**: CREATIVE_BRIEF.md + MASTER_BLUEPRINT.md

3. **Technical reference** (keep nearby):
   - **Frontend**: API_SPECIFICATION.md (what backend provides)
   - **Backend**: DATABASE_SCHEMA.md (what to store)
   - **Both**: AUTHENTICATION_GUIDE.md (how login works)

4. **When stuck**: DOCUMENTATION_INDEX.md
   - Navigation guide to all documents
   - Quick answers to common questions

---

## 👨‍💻 Frontend Developer - Day 1 Setup

### **Your mission**: Get "Hello Heralds!" showing in browser

**Step 1: Clone the repository** (5 mins)

```bash
# Open terminal/PowerShell
# Navigate to where you want the project
cd Documents

# Clone the code from GitHub
git clone https://github.com/allyzza010501-lab/UCC-student-portal.git

# Go into the project
cd UCC-student-portal/frontend

# Confirm you're in the right place - you should see:
# - src/
# - package.json
# - vite.config.ts
# - tailwind.config.ts
```

**Step 2: Install dependencies** (3 mins)

```bash
# Install all required packages
npm install

# This downloads React, Vite, Tailwind, etc.
# Takes 1-2 minutes - be patient!
```

**Step 3: Start the development server** (2 mins)

```bash
# Start the live development server
npm run dev

# You'll see something like:
# 
# VITE v5.0.0  ready in 234 ms
# ➜  Local:   http://localhost:5173/
# ➜  press h to show help
```

**Step 4: Open in browser** (1 min)

```
Click the link: http://localhost:5173
Or manually type it in your browser address bar

You should see the Vite default page!
```

**Step 5: First code change** (2 mins)

Edit `src/App.tsx`:
```typescript
// Find this line:
return <h1>Vite + React</h1>

// Change it to:
return <h1>Welcome to Heralds! 🎉</h1>

// Save the file - the browser updates automatically!
```

**Congratulations!** You just:
- ✅ Cloned code from GitHub
- ✅ Installed dependencies
- ✅ Started the dev server
- ✅ Made your first code change
- ✅ Saw Hot Module Replacement (HMR) in action

**If you got stuck**: Ask in the team chat! This is normal.

---

## 🔧 Backend Developer - Day 1 Setup

### **Your mission**: Get Cloudflare Workers running locally

**Step 1: Clone the repository** (same as frontend)

```bash
cd Documents
git clone https://github.com/allyzza010501-lab/UCC-student-portal.git
cd UCC-student-portal/backend
```

**Step 2: Install dependencies** (3 mins)

```bash
npm install

# This installs wrangler (Cloudflare CLI) and other packages
```

**Step 3: Authenticate with Cloudflare** (2 mins)

```bash
# This opens your browser to login to Cloudflare
npm run auth

# Or manually:
wrangler login

# Click "Allow" in the browser that opens
# Return to terminal when done
```

**Step 4: Create local database** (1 min)

```bash
# Create SQLite database for development
wrangler d1 create ucc-student-portal --local

# You'll see:
# ✅ Successfully created DB "ucc-student-portal"
```

**Step 5: Start development server** (2 mins)

```bash
# Start the local Cloudflare Workers server
npm run dev

# You'll see:
# 
# ⛅ wrangler 3.x.x
# ⎔ Using D1 database ucc-student-portal
# 🎉 Listening on http://localhost:8787
```

**Step 6: Test the API** (3 mins)

In another terminal:
```bash
# Test the health check endpoint
curl http://localhost:8787/api/health

# You should see:
# {"status":"ok","message":"Server is running"}
```

**Congratulations!** You just:
- ✅ Cloned code from GitHub
- ✅ Installed dependencies
- ✅ Authenticated with Cloudflare
- ✅ Created a local database
- ✅ Started the API server
- ✅ Made your first API request

**If you got stuck**: Ask in the team chat!

---

## 🎨 Designer - Day 1 Setup

### **Your mission**: Understand the visual system and create first design

**Step 1: Read the design documents** (20 mins)

1. **CREATIVE_BRIEF.md** (10 mins)
   - Brand direction
   - Color palette
   - Typography
   - Visual principles

2. **MASTER_BLUEPRINT.md Design System section** (10 mins)
   - Component list
   - Layout patterns
   - Responsive breakpoints

**Step 2: Set up your design tool** (15 mins)

Option A: **Figma** (recommended, free)
- Go to figma.com
- Create account
- Create new design file named "Heralds Design System"
- Set color palette:
  - Primary: #0066CC (blue)
  - Secondary: #FF6B35 (orange)
  - Accent: #2ECC71 (green)
  - Warning: #FFC107 (gold)

Option B: **Adobe XD** or **Sketch** (if you have access)

**Step 3: Create first page** (15 mins)

Create a "Design System" page showing:
- [ ] Color palette with all 4 colors + variations
- [ ] Typography: Headings, body text, captions
- [ ] Basic components: Button, card, input field
- [ ] Responsive grid (mobile, tablet, desktop)

**Step 4: Share with team** (5 mins)

If using Figma:
- Click "Share"
- Give edit access to: [team members' emails]
- Send link in team chat

**Step 5: Attend design standup** (10 mins)

Review design with team:
- Is the direction clear?
- Are colors right?
- Any questions?

**Your first deliverable**: Design System page (due Wednesday)

---

## 🧪 QA/Tester - Day 1 Setup

### **Your mission**: Understand what we're testing and create test plan

**Step 1: Read feature specifications** (20 mins)

- **FEATURE_SPECIFICATIONS.md**
- Understand all 8 MVP features
- Read acceptance criteria for each

**Step 2: Set up testing tools** (15 mins)

You'll need:
- Browser for manual testing (Chrome, Firefox, Safari)
- Notepad or Confluence for test cases
- Optional: Postman (for API testing)

**Step 3: Create test plan skeleton** (20 mins)

Create a document with:

```
TEST PLAN - Heralds Student Portal

1. USER REGISTRATION TESTING
   - Test case 1.1: Valid registration
   - Test case 1.2: Invalid email
   - Test case 1.3: Weak password
   - Test case 1.4: Email already exists
   
2. LOGIN TESTING
   - Test case 2.1: Valid login
   - Test case 2.2: Invalid password
   - Test case 2.3: Rate limiting (5 attempts)
   
3. FEED TESTING
   ... (etc)
```

**Step 4: Join dev team** (10 mins)

Attend development standup:
- Understand what's being built
- Understand testing priorities
- Ask questions

**Your first deliverable**: Complete test plan (due Friday)

---

## 🎯 Everyone - Immediate Next Steps

### **This Week** (Dec 16-20)

**Monday (Today)**:
- [ ] 9 AM: Team standup (all together)
- [ ] Get local dev environment running
- [ ] Read your role-specific setup guide
- [ ] Ask questions in team chat
- [ ] Celebrate! 🎉

**Tuesday-Wednesday**:
- [ ] Frontend dev: Build login form component
- [ ] Backend dev: Build /auth/register endpoint
- [ ] Designer: Design login page mockup
- [ ] Tester: Create login test cases

**Thursday**:
- [ ] Demo what you built to team
- [ ] Code review (pair programming)
- [ ] Iterate based on feedback

**Friday**:
- [ ] 9 AM: Sprint review (what we built)
- [ ] 10 AM: Retrospective (what we learned)
- [ ] 11 AM: Plan next week
- [ ] Celebrate end of week! 🎉

### **This Month** (Dec 16 - Jan 20)

| Week | Focus | Deliverable |
|------|-------|-------------|
| Week 1-2 | Design system & components | Reusable component library |
| Week 3-4 | Landing page | Beautiful homepage |
| Week 5-6 | Core features | Posts, comments, reactions |
| Week 7-8 | Polish & launch | Live on ucc-student-portal.com |

---

## 🆘 What If I Get Stuck?

### **Problem: Code won't compile**

**Solution**: 
1. Check error message - Google the exact error
2. Ask in team chat with:
   - What you're trying to do
   - The exact error message
   - What you've already tried
3. I'll help debug

### **Problem: Can't install dependencies**

**Solution**:
1. Delete `node_modules/` folder
2. Delete `package-lock.json` file
3. Run `npm install` again
4. If still stuck, ask in team chat

### **Problem: Dev server won't start**

**Solution**:
1. Make sure port isn't used: `lsof -i :5173` (Mac/Linux) or check Task Manager (Windows)
2. Kill any existing process using that port
3. Run `npm run dev` again

### **Problem: Browser shows blank page**

**Solution**:
1. Open browser console: F12 or Cmd+Option+J
2. Look for red error messages
3. Screenshot the error and ask in team chat

### **Problem: Can't understand a document**

**Solution**:
1. Ask me directly - explain what's confusing
2. I'll rephrase it in simpler terms
3. Documentation exists to help you, not confuse you!

---

## 💬 Communication Expectations

### **Daily Standup** (10 mins, 9 AM)
**Format**: 
- "Yesterday I did..." (1 min)
- "Today I'll do..." (1 min)
- "I'm blocked by..." (if applicable)

**Why**: Keep everyone aware of progress and blockers

### **Code Review** (before merging code)
**Format**:
- Show your code in chat/GitHub
- Another person reviews it
- Discuss improvements
- Merge when approved

**Why**: Catch bugs early, keep code quality high

### **Weekly Retrospective** (Friday, 10 AM)
**Format**:
- What went well?
- What was hard?
- What should we improve?

**Why**: Continuous improvement - we get better every week

### **In Team Chat**
Feel free to ask:
- ✅ Technical questions
- ✅ How something works
- ✅ Clarification on requirements
- ✅ Help with blockers
- ✅ Celebration when you ship something!

**Rule**: No such thing as "dumb question" - we're all learning!

---

## 📞 Key Contacts

| Role | Name | Email | What they do |
|------|------|-------|-------------|
| Project Lead | [Your name] | [Your email] | Approvals, decisions, UCC communication |
| AI Co-Pilot | Me | (in chat) | Daily support, code help, documentation |
| Lead Frontend | [Name] | [Email] | Frontend architecture, code review |
| Lead Backend | [Name] | [Email] | Backend architecture, API design |
| Lead Designer | [Name] | [Email] | Design system, mockups |
| QA Lead | [Name] | [Email] | Test plans, bug reports |

---

## 🎓 Learning Resources

If you need to learn something new:

| Technology | Resource | Time |
|------------|----------|------|
| React Hooks | react.dev (official) | 2 hours |
| TypeScript Basics | typescriptlang.org | 1 hour |
| Tailwind CSS | tailwindcss.com | 1 hour |
| REST APIs | mdn.org | 1 hour |
| Cloudflare Workers | developers.cloudflare.com | 2 hours |
| Git/GitHub | github.com/skills | 1 hour |

**Don't have time to learn?** That's OK! We'll learn together as we build.

---

## 🎉 First Week Goals

By end of Friday (Dec 20), celebrate if you've:

✅ Got local dev environment running  
✅ Made your first code change (frontend) or endpoint (backend)  
✅ Read all your role-specific documentation  
✅ Attended daily standups  
✅ Asked at least one question (shows you're engaged!)  
✅ Helped a teammate (shows you care!)  
✅ Built something that works (even if it's small)  

If you hit all these, you've had an amazing first week! 🚀

---

## ⚠️ Things NOT to Do

❌ Don't skip reading the documentation  
❌ Don't be shy about asking questions  
❌ Don't merge code without code review  
❌ Don't commit to passwords or secrets  
❌ Don't skip the standup  
❌ Don't ignore error messages - they're helpful!  
❌ Don't be afraid to break things - that's how we learn!  

---

## ✨ Things TO Do

✅ Ask questions - that's your job!  
✅ Help teammates - we're a team!  
✅ Test your own code first  
✅ Read error messages carefully  
✅ Celebrate small wins  
✅ Share what you learn  
✅ Have fun building Heralds!  

---

## 🏁 Let's Go! 

**Monday is here. The plans are made. The team is ready.**

**Your first task at 9 AM**: 
Show up to standup with excitement, not stress.

**Your second task**:
Get your dev environment running.

**Everything else**:
We'll figure it out together, one step at a time.

---

## 📋 Quick Checklist (Copy This & Check Off as You Go)

**Before 9 AM Monday**:
- [ ] Node.js installed and working
- [ ] Git configured
- [ ] GitHub account active
- [ ] Code editor ready
- [ ] This guide printed/saved
- [ ] Coffee/water ready (you'll need it!)

**9 AM Monday - After standup**:
- [ ] Clone repository
- [ ] Install dependencies
- [ ] Start dev server
- [ ] See it running in browser
- [ ] Make first code change
- [ ] High five a teammate! 🙌

**End of Monday**:
- [ ] Understand the project
- [ ] Know what you're building this week
- [ ] Have team members' contact info
- [ ] Know where to ask for help
- [ ] Feel excited about Heralds!

---

**See you Monday morning at 9 AM!** 

Let's build something amazing together! 🚀

🎉 **Welcome to Team Heralds!** 🎉

---

*This guide prepared by: AI Co-Pilot*  
*Date: December 11, 2025*  
*For: All team members joining Monday, Dec 16*
