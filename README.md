# IndyCar Official Training System

A comprehensive training platform for IndyCar officials featuring pit stop infraction analysis, instructional videos, and certification tracking.

## 🎯 Purpose

Transform raw pit stop footage captured via Meta glasses into structured training content for:
- New official onboarding
- Infraction recognition certification
- Scanner management training
- Pit wall safety protocols

## 📁 Repository Structure

```
├── docs/                    # Documentation
│   ├── SYSTEM_DESIGN.md     # Full architecture
│   ├── WORKFLOW.md          # Approval processes
│   └── TAGGING_GUIDE.md     # How to tag infractions
│
├── videos/                  # All video content
│   ├── pending-review/      # Incoming from Meta
│   ├── infractions/         # Approved infraction videos
│   ├── instructional/       # Training content
│   └── rejected/            # Rejected with reasons
│
├── data/                    # Metadata & configurations
│   ├── infraction-types.json
│   └── certification-tracks.json
│
├── src/                     # Source code
│   ├── ingestion/           # Video download automation
│   ├── tagging/             # Web-based tagging UI
│   └── portal/              # Training web app
│
└── scripts/                 # Automation scripts
    ├── setup.sh
    └── daily-sync.sh
```

## 🚀 Quick Start

### For Officials (Training)
1. Visit the training portal: [Link TBD]
2. Complete certification tracks
3. Review infraction library

### For Content Creators (Mark)
1. Record with Meta glasses → auto-uploads
2. Review approval notifications
3. Tag approved videos with timestamps

## 🔄 Workflow

```
Meta Glasses → Cloud → Download → Review → Approve → Tag → Train
```

## 📋 Status

- [x] Repository created
- [ ] Automation setup
- [ ] Approval workflow
- [ ] Tagging interface
- [ ] Training portal

## 📞 Contact

- **Owner:** Mark Glotzbach
- **System Design:** See [docs/SYSTEM_DESIGN.md](docs/SYSTEM_DESIGN.md)

---

**Private Repository** - Authorized IndyCar personnel only
