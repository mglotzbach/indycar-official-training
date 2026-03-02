# IndyCar Official Training System
## Comprehensive Design Document

**Version:** 1.0  
**Date:** March 2, 2026  
**Owner:** Mark Glotzbach  
**Status:** Architecture & Implementation Proposal

---

## Executive Summary

This system transforms raw pit stop footage captured via Meta glasses into a structured training platform for IndyCar officials. The workflow includes:

1. **Automated ingestion** from Meta glasses
2. **Approval workflow** with IndyCar Officiating
3. **Timestamp-based tagging** of infractions
4. **Training portal** with certification tracking
5. **Instructional library** for onboarding

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     DATA SOURCES                                │
├─────────────────────────────────────────────────────────────────┤
│  Meta Glasses → Mobile App → Meta Cloud → Download Agent        │
│  (WiFi/Cellular auto-upload)         (Automated fetch)          │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                     INGESTION PIPELINE                          │
├─────────────────────────────────────────────────────────────────┤
│  1. Video downloaded to `/videos/pending-review/`               │
│  2. Auto-generate metadata (timestamp, duration, file size)     │
│  3. Create review issue in GitHub                               │
│  4. Notification sent to Mark                                   │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                     APPROVAL WORKFLOW                           │
├─────────────────────────────────────────────────────────────────┤
│  Status: PENDING → UNDER_REVIEW → APPROVED/REJECTED            │
│                                                                  │
│  Approvers:                                                      │
│  - Mark Glotzbach (Primary)                                     │
│  - IndyCar Officiating Representative                           │
│                                                                  │
│  Actions:                                                        │
│  - Approve → Move to `/videos/infractions/` or `/instructional/`│
│  - Reject → Move to `/videos/rejected/` with reason             │
│  - Request Edit → Back to pending with comments                 │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                     TAGGING & METADATA                          │
├─────────────────────────────────────────────────────────────────┤
│  Web interface for timestamp tagging:                           │
│  - Infraction type (dropdown)                                   │
│  - Severity (Minor/Major/Critical)                              │
│  - Timestamp (auto-pause video at mark)                         │
│  - Notes/description                                            │
│  - Related rules/regulations                                    │
│                                                                  │
│  Output: JSON sidecar file per video                            │
│  Example: `pitstop_001_20260302.json`                           │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                     TRAINING PORTAL                             │
├─────────────────────────────────────────────────────────────────┤
│  Features:                                                       │
│  - Quiz mode: "What infraction occurred at 00:32?"              │
│  - Reference library: Search by infraction type                 │
│  - Progress tracking: Certification completion                  │
│  - Instructional videos: Scanner usage, wall protocols          │
│  - Discussion threads: Per-video Q&A                            │
└─────────────────────────────────────────────────────────────────┘
```

---

## Repository Structure

```
indycar-official-training/
├── README.md
├── docs/
│   ├── SYSTEM_DESIGN.md              # This document
│   ├── WORKFLOW.md                   # Approval process details
│   ├── TAGGING_GUIDE.md              # How to tag infractions
│   ├── API_REFERENCE.md              # Automation endpoints
│   └── ONBOARDING.md                 # New official guide
│
├── videos/
│   ├── pending-review/               # Incoming from Meta
│   │   └── .gitkeep
│   ├── infractions/                  # Approved infraction videos
│   │   ├── 2026-season/
│   │   └── metadata/                 # JSON sidecars
│   ├── instructional/                # Training content
│   │   ├── scanner-usage/
│   │   ├── wall-protocols/
│   │   ├── pit-equipment/
│   │   └── emergency-procedures/
│   ├── rejected/                     # Rejected with reasons
│   │   └── rejection-log.md
│   └── archived/                     # Season-end archive
│
├── data/
│   ├── infraction-types.json         # Controlled vocabulary
│   ├── certification-tracks.json     # Learning paths
│   ├── user-progress.json            # Completion tracking
│   └── video-index.json              # Searchable index
│
├── src/
│   ├── ingestion/                    # Meta → Repo automation
│   │   ├── meta_download.py
│   │   └── video_processor.py
│   ├── tagging/                      # Timestamp tagging UI
│   │   ├── tagger.html
│   │   └── tagger.js
│   ├── portal/                       # Training web app
│   │   ├── index.html
│   │   ├── quiz-engine.js
│   │   └── progress-tracker.js
│   └── approval/                     # Workflow automation
│       ├── approval-bot.py
│       └── notification.py
│
├── scripts/
│   ├── setup.sh                      # Initial setup
│   ├── daily-sync.sh                 # Cron: Fetch new videos
│   └── generate-report.sh            # Weekly stats
│
└── .github/
    ├── workflows/
    │   ├── video-processing.yml      # GitHub Actions
    │   └── approval-reminders.yml
    ├── ISSUE_TEMPLATE/
    │   └── video-review.md
    └── CODEOWNERS                    # Approval requirements
```

---

## Phase 1: Video Ingestion Automation

### Current State
- Meta glasses record pit stops
- Videos auto-upload to Meta Cloud
- Manual download to computer
- Manual organization

### Proposed Automation

#### Option A: Meta Cloud API Integration (Preferred)

**Setup:**
1. Create Meta developer account
2. Register app for Cloud API access
3. OAuth consent for `mglotzbach@gmail.com`
4. Store tokens securely (GitHub Secrets)

**Workflow:**
```python
# meta_download.py (runs every 15 minutes)
1. Authenticate with Meta Cloud API
2. List new videos since last check
3. Download to `videos/pending-review/`
4. Generate metadata JSON:
   {
     "source": "meta-glasses",
     "recorded_at": "2026-03-02T14:32:00Z",
     "duration": 45.2,
     "file_size_mb": 128.5,
     "auto_tags": ["pit-stop", "auto-upload"],
     "status": "pending-review"
   }
5. Create GitHub Issue for review
6. Send Telegram notification to Mark
```

**Pros:**
- Fully automated
- Real-time (15-min latency)
- Direct from source

**Cons:**
- Requires Meta Cloud API access (may need approval)
- API rate limits

#### Option B: Google Photos Integration (Fallback)

**Setup:**
1. Meta glasses sync to Google Photos
2. Agent monitors Google Photos album
3. Downloads new videos automatically

**Workflow:**
```python
# google_photos_monitor.py
1. Poll Google Photos API every 15 minutes
2. Detect new videos in "IndyCar Pit Stops" album
3. Download to `videos/pending-review/`
4. Same metadata generation as Option A
```

**Pros:**
- Google Photos already likely in use
- Simpler API access

**Cons:**
- Extra sync step (Meta → Google Photos)
- Slight delay in chain

### Recommendation

**Start with Option B (Google Photos)** because:
1. Faster to implement (no Meta developer approval)
2. You likely already use Google Photos
3. Can migrate to Option A later if needed

**Migration path:** Build abstraction layer so switching is 1-line change.

---

## Phase 2: Approval Workflow

### GitHub-Based Workflow

**Why GitHub:**
- Native approval mechanisms (CODEOWNERS, PR reviews)
- Issue tracking for each video
- Audit trail (who approved what when)
- Integration with automation

**Process:**

1. **Video Arrives** → Auto-creates GitHub Issue
   ```markdown
   Title: [PENDING] Pit Stop Video - 2026-03-02 14:32
   
   ## Video Details
   - File: `pitstop_20260302_143200.mp4`
   - Duration: 45s
   - Auto-detected tags: pit-stop, turn-4
   
   ## Review Checklist
   - [ ] Contains clear infraction OR instructional value
   - [ ] Audio quality acceptable
   - [ ] No sensitive info visible
   - [ ] Approved by IndyCar Officiating
   
   ## Actions
   @mglotzbach please review
   
   /cc @indycar-officiating-rep
   ```

2. **Review Phase**
   - Mark reviews video
   - Adds comments if issues
   - Requests changes or approves

3. **Approval**
   - Mark: `/approve infraction` or `/approve instructional`
   - Bot moves file to correct folder
   - Closes issue
   - Notifies next step (tagging)

4. **Rejection**
   - Mark: `/reject [reason]`
   - Bot moves to `rejected/`
   - Logs reason in `rejection-log.md`

### Automation Bot

```python
# approval-bot.py (GitHub Actions)

on: issue_comment

if comment starts with "/approve":
  1. Parse video file from issue
  2. Move from pending-review/ to approved folder
  3. Update metadata status
  4. Close issue
  5. Create new issue: "[TAGGING REQUIRED] Video ready for tagging"

if comment starts with "/reject":
  1. Parse reason
  2. Move to rejected/
  3. Log reason
  4. Close issue with rejection label
```

---

## Phase 3: Timestamp Tagging Framework

### Design Goals
- **Fast:** Tag 30-second video in <2 minutes
- **Precise:** Frame-accurate timestamps
- **Consistent:** Controlled vocabulary
- **Portable:** JSON export for any platform

### Web Interface

**Tech Stack:**
- Frontend: Vanilla JS (no build step needed)
- Hosting: GitHub Pages (free, integrated)
- Video: HTML5 video element
- Storage: JSON files in repo

**Interface Mockup:**
```
┌─────────────────────────────────────────────────────────────┐
│ IndyCar Infraction Tagger                    [Save] [Exit] │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────┐                 │
│  │                                         │                 │
│  │         VIDEO PLAYER                    │                 │
│  │         (with frame controls)           │                 │
│  │                                         │                 │
│  └─────────────────────────────────────────┘                 │
│                                                              │
│  [<<] [Play/Pause] [>>]        Time: 00:32.145 / 00:45.200  │
│                                                              │
├─────────────────────────────────────────────────────────────┤
│ NEW TAG                                                      │
│                                                              │
│ Timestamp: [00:32.145]  [Capture Current]                   │
│                                                              │
│ Infraction Type: [Tire Change Violation ▼]                  │
│   ├─ Tire Change Violation                                  │
│   ├─ Unsafe Release                                         │
│   ├─ Equipment Interference                                 │
│   ├─ Personnel Outside Box                                  │
│   ├─ Fueling Violation                                      │
│   └─ Other (specify)                                        │
│                                                              │
│ Severity: (•) Minor  ( ) Major  ( ) Critical               │
│                                                              │
│ Description:                                                 │
│ [Front tire carrier stepped outside box boundary          ] │
│ [before tire was secured                                   ] │
│                                                              │
│ Rule Reference: [Rule 14.3.2 ▼]                            │
│                                                              │
│ [+ Add Tag]                                                 │
├─────────────────────────────────────────────────────────────┤
│ EXISTING TAGS (2)                                            │
│ • 00:12.400 - Unsafe Release (Major)                        │
│ • 00:32.145 - Tire Change Violation (Minor) ← Editing       │
└─────────────────────────────────────────────────────────────┘
```

### Metadata Schema

```json
{
  "video_id": "pitstop_20260302_143200",
  "video_file": "infractions/2026-season/pitstop_20260302_143200.mp4",
  "metadata": {
    "recorded_at": "2026-03-02T14:32:00Z",
    "location": "Turn 4 Pit Box",
    "event": "Firestone Grand Prix of St. Petersburg",
    "session": "Race",
    "uploaded_by": "Mark Glotzbach",
    "approved_by": "IndyCar Officiating",
    "approved_at": "2026-03-02T18:45:00Z"
  },
  "tags": [
    {
      "id": "tag_001",
      "timestamp": 12.4,
      "timestamp_formatted": "00:12.400",
      "infraction_type": "unsafe_release",
      "infraction_label": "Unsafe Release",
      "severity": "major",
      "description": "Car released into path of incoming traffic",
      "rule_reference": "Rule 14.7.1",
      "created_by": "Mark Glotzbach",
      "created_at": "2026-03-02T19:30:00Z"
    },
    {
      "id": "tag_002",
      "timestamp": 32.145,
      "timestamp_formatted": "00:32.145",
      "infraction_type": "tire_change_boundary",
      "infraction_label": "Tire Change Violation",
      "severity": "minor",
      "description": "Front tire carrier stepped outside box boundary",
      "rule_reference": "Rule 14.3.2",
      "created_by": "Mark Glotzbach",
      "created_at": "2026-03-02T19:32:00Z"
    }
  ],
  "summary": {
    "total_infractions": 2,
    "severity_breakdown": {
      "critical": 0,
      "major": 1,
      "minor": 1
    },
    "processing_status": "ready_for_training"
  }
}
```

### Controlled Vocabulary

```json
{
  "infraction_types": [
    {
      "id": "unsafe_release",
      "label": "Unsafe Release",
      "description": "Car released into dangerous conditions",
      "typical_severity": "major",
      "rule_reference": "Rule 14.7.1"
    },
    {
      "id": "tire_change_boundary",
      "label": "Tire Change Violation",
      "description": "Personnel outside pit box during tire change",
      "typical_severity": "minor",
      "rule_reference": "Rule 14.3.2"
    },
    {
      "id": "equipment_interference",
      "label": "Equipment Interference",
      "description": "Equipment contacting car or personnel",
      "typical_severity": "major",
      "rule_reference": "Rule 14.5.3"
    },
    {
      "id": "fueling_violation",
      "label": "Fueling Violation",
      "description": "Unsafe fueling practices",
      "typical_severity": "critical",
      "rule_reference": "Rule 14.9.1"
    },
    {
      "id": "personnel_count",
      "label": "Personnel Count Violation",
      "description": "Too many personnel over wall",
      "typical_severity": "minor",
      "rule_reference": "Rule 14.2.1"
    }
  ]
}
```

---

## Phase 4: Training Portal

### Features

1. **Learning Tracks**
   - New Official Certification (10 videos + quiz)
   - Wall Worker Safety (5 videos)
   - Infraction Recognition (20 videos)
   - Scanner Management (3 instructional videos)

2. **Quiz Engine**
   - Show video with timestamp
   - "What infraction occurred?"
   - Multiple choice with immediate feedback
   - Reference to rule book

3. **Progress Tracking**
   - Individual completion %
   - Track by certification track
   - Manager dashboard (who's certified)

4. **Reference Library**
   - Search: "Show me all unsafe releases"
   - Filter by severity, date, event
   - Side-by-side comparison

### Sample Quiz

```json
{
  "quiz_id": "infraction_recognition_001",
  "title": "Pit Stop Infraction Recognition - Set 1",
  "questions": [
    {
      "question_id": "q_001",
      "type": "video_identification",
      "video_file": "infractions/2026-season/pitstop_20260302_143200.mp4",
      "pause_at": 12.4,
      "question": "What infraction is occurring at this moment?",
      "options": [
        "Unsafe Release",
        "Tire Change Violation",
        "Equipment Interference",
        "No Infraction"
      ],
      "correct_answer": 0,
      "explanation": "The car was released into the path of incoming traffic. See Rule 14.7.1",
      "reference_video": "instructional/unsafe-release-examples.mp4"
    }
  ]
}
```

---

## Implementation Roadmap

### Week 1: Foundation
- [ ] Set up repository structure
- [ ] Create GitHub Actions for automation
- [ ] Implement Meta/Google Photos download script
- [ ] Basic approval workflow

### Week 2: Ingestion
- [ ] Automate video download (cron every 15 min)
- [ ] Auto-generate metadata
- [ ] GitHub issue creation for review
- [ ] Telegram notifications

### Week 3: Tagging Interface
- [ ] Build web-based tagger (GitHub Pages)
- [ ] Implement timestamp capture
- [ ] Create controlled vocabulary JSON
- [ ] Save/load metadata

### Week 4: Training Portal
- [ ] Quiz engine
- [ ] Progress tracking
- [ ] Learning tracks
- [ ] Reference library search

### Week 5: Polish
- [ ] IndyCar Officiating review & feedback
- [ ] Documentation
- [ ] Onboard first batch of officials

---

## Security & Privacy

1. **Video Access**
   - Repository: Private (invite-only)
   - IndyCar Officiating: Read access
   - Individual officials: Access to approved training content only

2. **Sensitive Content**
   - No driver personal info in metadata
   - No team strategy visible
   - Focus on officiating mechanics only

3. **Backup**
   - Videos backed up to Google Drive (sync script)
   - Metadata in Git (version controlled)
   - Disaster recovery plan

---

## Next Steps

1. **Review this proposal** with IndyCar Officiating
2. **Decide:** Google Photos vs Meta Cloud API
3. **Grant access:** Add IndyCar Officiating rep to repo
4. **Begin:** Week 1 implementation

**Questions to resolve:**
- Who is the IndyCar Officiating representative for approvals?
- Do you already use Google Photos with Meta glasses?
- What infraction types should be in the initial vocabulary?

---

**Repo Created:** https://github.com/mglotzbach/indycar-official-training

Ready to begin implementation! 🏎️
