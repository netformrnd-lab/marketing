# CLAUDE.md — AI Assistant Guide for Marketing Workspace

## Project Overview

This is a **single-file React application** (`index.html`, ~18,000 lines) serving as a team workspace for the **Brand Marketing Team (브랜드마케팅팀)** at NetForm R&D. The entire application — styles, components, data models, and logic — lives in one HTML file using CDN-loaded React 18 with Babel for in-browser JSX compilation.

**Language**: All UI text is in Korean (ko). Commit messages are also in Korean.

## Architecture

### Single-File Structure

```
marketing/
└── index.html    # The entire application (HTML + CSS + React/JSX)
```

There is no build system, no bundler, no `package.json`. The app runs directly in the browser via CDN scripts and Babel standalone.

### CDN Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| React | 18 (UMD) | UI framework |
| ReactDOM | 18 (UMD) | DOM rendering |
| Babel Standalone | latest | In-browser JSX compilation |
| Firebase | 10.7.1 (compat) | Auth, Realtime DB, Storage |
| html2canvas | 1.4.1 | Screenshot/image export |
| jsPDF | 2.5.1 | PDF generation |
| Pretendard | latest | Korean web font |

### State Management

Pure React `useState`/`useEffect` hooks in the root `TeamWorkApp` component. No Redux or external state libraries. Key state is synced bidirectionally with Firebase Realtime Database via `onDataChange` listeners.

### Persistence

- **Firebase Realtime Database**: All shared data (projects, tasks, meetings, KPI, team members, settings)
- **localStorage**: User session, read comments, favorite menus, daily notes

## Key Components & Navigation

The app uses **tab-based navigation** (no URL router). The `activeTab` state controls which view renders.

### Component Map (approximate line numbers)

| Component | ~Line | Purpose |
|-----------|-------|---------|
| `Icon` | 150 | SVG icon system (25+ types) |
| `FirebaseService` | 303 | Database CRUD & real-time sync |
| `LoginScreen` | 471 | Email/password authentication |
| `TeamWorkApp` | 744 | Root app, navigation, all state |
| `AccountManageTab` | 1726 | Team member account management |
| `TemplateManageTab` | 2129 | Project template editor |
| `MeetingTab` | 2444 | Meeting minutes & action items |
| `ManualTab` | 3557 | Onboarding documentation |
| `KPIDashboardTab` | 6229 | KPI tracking & metrics |
| `PaymentCycleTab` | 6671 | Payment schedule tracking |
| `HomeTab` | 7615 | Dashboard overview |
| `TeamRequestTab` | 7827 | Cross-team task requests |
| `WorkLogTab` | 8650 | Daily work log & calendar |
| `ProjectTab` | 11534 | Project management |
| `ProjectDetailSlide` | 14745 | Project detail slide-over panel |
| `CalendarTab` | 16317 | Monthly calendar view |
| `TrashTab` | 16894 | Deleted items (7-day auto-delete) |
| `GrowthBoardTab` | 17219 | Interactive canvas/board |

### Navigation Structure

```
홈 (Home)
업무관리 (Work Management)
  ├── 업무일지 (Work Log)
  ├── 프로젝트 (Projects)
  └── 타팀요청 업무 (Team Requests)
성과관리 (Performance)
  ├── 그로스보드 (Growth Board)
  └── KPI 대시보드 (KPI Dashboard)
지식베이스 (Knowledge Base)
  ├── 온보딩/매뉴얼 (Onboarding/Manual)
  ├── 회의록 (Meetings)
  └── 드라이브 (File Storage — placeholder)
관리 (Management)
  ├── 템플릿 관리 (Templates)
  ├── 결제 주기 체크 (Payment Tracking)
  ├── 업체 관리 (Vendor — placeholder)
  └── 계정 관리 (Accounts)
```

## Data Models

### Core Entities

**Brands** (`BRANDS` constant):
넷폼알앤디, POUR공법, POUR솔루션, 아파트스퀘어, 자동화&시스템화, 석민이앤씨

**Team Members** (`TEAM_MEMBERS` constant):
5 members with id, name, initial, role (팀장/매니저), email, isAdmin flag.

**Task**: `{id, projectId, title, memberId, date, completed, order, note, bookmark, priority, parentTaskId, deletedAt}`

**Project**: `{id, brandId, title, description, status, createdDate, creatorId, memberIds, comments, linkedKpi, linkedKpis, deletedAt}`

**Meeting**: `{id, title, date, startTime, endTime, content, attendees, actionItems, createdBy, linkedProjects}`

**Team Request**: `{id, title, content, requestTeam, brandId, priority, assigneeId, status, dueDate}`

### Firebase Database Paths

```
projects/          tasks/             fixedTasks/
teamRequests/      meetings/          teamMembers/
kpi/companyGoal    kpi/teamGoal       kpi/brands/{brandId}/
kpi/teamMembers/   kpi/quarterPlans/  kpi/quarterGoals/
settings/projectTemplates             settings/dailyNotes
```

## Development Conventions

### Commit Messages

- Written in **Korean**
- Describe the change concisely (e.g., `결제 항목 폼 정리: 결제수단 제거, 계약정보 섹션에 다음 결제일 통합`)
- Use colon to separate scope from detail when helpful

### Code Style

- **Inline styles**: Most styling is done via inline `style={{}}` objects, not CSS classes
- **Global CSS**: Defined in `<style>` block at the top of the file for animations, print styles, scrollbar, and reusable class-based styles (`.kpi-*`, `.task-row`, etc.)
- **No TypeScript** — plain JavaScript/JSX
- **Korean variable naming**: Some constants use Korean text in values; variable/function names are in English
- **Date format**: ISO `YYYY-MM-DD` throughout

### Editing Guidelines

Since the entire app is a single 18,000+ line file:

1. **Use line numbers** — Always reference approximate line numbers when navigating
2. **Search by component name** — Components are defined as `function ComponentName(` or `const ComponentName =`
3. **Be cautious with edits** — A syntax error anywhere breaks the entire app
4. **Test scope** — There are no automated tests; changes must be verified manually in-browser
5. **Preserve Firebase paths** — Changing database paths will break data sync for all users

### Special URL Parameters

- `?view=kpi` — Public KPI dashboard view (no authentication required)

### Print Support

CSS `@media print` rules are defined for optimized printing of calendars and work logs. The sidebar and headers are hidden in print mode.

## Firebase Configuration

- **Project**: `mktt-764a6`
- **Auth**: Email/password (hardcoded `@workspace.local` domain)
- **Database**: Asia Southeast region (`https://mktt-764a6-default-rtdb.asia-southeast1.firebasedatabase.app`)
- **Storage**: Default bucket

The `FirebaseService` object (around line 303) provides:
- `getAll(path)` — Fetch collection as array
- `save(path, data)` — Upsert document
- `delete(path)` — Remove document
- `onDataChange(path, callback)` — Real-time listener
- `getDoc(path)` / `saveDoc(path, data)` — Single document ops

## Common Tasks

### Adding a new menu/tab

1. Add a menu item entry in the sidebar render section of `TeamWorkApp`
2. Create the new tab component function
3. Add a case in the tab rendering conditional inside `TeamWorkApp`

### Adding a new data entity

1. Define the Firebase path
2. Add `useState` in `TeamWorkApp`
3. Set up `onDataChange` listener in the data-loading `useEffect`
4. Pass state + save handlers as props to the relevant tab component

### Modifying brand or team member data

Update the `BRANDS` or `TEAM_MEMBERS` constants near the top of the `<script>` block. Note that `TEAM_MEMBERS` is supplemented at runtime by dynamic entries from `teamMembers/` in Firebase.
