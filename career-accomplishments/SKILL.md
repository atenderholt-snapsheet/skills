---
name: career-accomplishments
description: Track career accomplishments tied to core values and competencies. Use this when the user wants to log an accomplishment, do a weekly reflection, or record their wins—anything involving capturing and cataloging work that demonstrates progress toward their goals. This skill helps you reflect on impact, classify accomplishments against your company's core values and role competencies, and builds a searchable record organized by quarter and week.
compatibility: Markdown, file system access
---

# Career Accomplishments Tracker

This skill helps you capture, reflect on, and organize career accomplishments. It works in two modes:

1. **Specific Accomplishment** — You describe something you did, and the skill asks clarifying questions to understand impact, collaborators, and timeline, then classifies it against your core values and competencies.

2. **Weekly Reflection** — The skill asks guided questions about your week to surface wins you may not have thought to log.

Both modes save accomplishments to a structured archive organized by quarter and week.

## How to use

### Mode A: Log a specific accomplishment

You can say things like:
- "I just refactored the authentication module to use JWT tokens"
- "We shipped the new onboarding flow"
- "Mentored Sarah through her first major project"

The skill will:
1. Ask clarifying questions if details are sparse:
   - **Impact**: What was the outcome? Who benefited? If quantifiable, what's the metric?
   - **Collaborators & Roles**: Who did you work with? What were their roles/titles?
   - **Timeline**: When did this happen? How long did it take?
2. Ask you to select which **Core Values** this accomplishment demonstrates (from your company's 6 core values)
3. Ask you to select which **Competencies** this demonstrates (from your role's framework)
4. Save the accomplishment to the appropriate week's file with all metadata

### Mode B: Weekly reflection

You can say:
- "Let's do my weekly reflection"
- "What did I accomplish this week?"
- "Sync my week to accomplishments"

The skill will:
1. Ask guided reflection questions about your week:
   - Key projects or tasks you worked on
   - Moments you felt you had impact
   - Times you collaborated or influenced others
   - Things you learned or improved
   - Any mentoring or leadership moments
2. Help you extract accomplishments from your reflection
3. Classify each one against core values and competencies
4. Save them to this week's file

## Storage structure

Accomplishments are stored in `~/Documents/Career/accomplishments/`:

```
accomplishments/
├── README.md                 (TOC, quarterly summaries)
├── 2026Q3/
│   ├── week_36/
│   │   └── README.md         (accomplishments for that week)
│   ├── week_37/
│   │   └── README.md
│   └── ...
└── 2026Q4/
    └── week_40/
        └── README.md
```

Each week's file contains:
- A list of accomplishments with metadata (date, time spent, collaborators, impact)
- Tags for core values and competencies
- Metadata for searching/summarizing later

The root README maintains:
- Table of contents linking to each quarter
- Quarterly summaries (key themes, major wins, growth areas)
- Statistics (accomplishments per quarter, most-cited core values/competencies)

## Accomplishment format

Each accomplishment in a week's file looks like:

```markdown
### [Title of accomplishment]

**Date**: 2026-08-21  
**Time spent**: 3 days (effort estimate)  
**Collaborators**: [Names and roles]

**Description**:
[What you did and why it mattered]

**Impact**:
- [Quantifiable outcome, if applicable]
- [Who benefited]
- [Scope or reach]

**Core Values**: [Value1, Value2]  
**Competencies**: [Competency1, Competency2]
```

## How the skill works

1. **Determine context**: Figure out the current quarter and week (or ask if the accomplishment is from a different week)
2. **Gather details**: Ask clarifying questions for incomplete information (impact, collaborators, timeline)
3. **Classify**: Help you tag core values and competencies by asking which ones resonate most
4. **Save**: Create or update the week's markdown file
5. **Update summaries**: Offer to regenerate the quarterly summary and root README (TOC)

## Tips for best results

- Be specific about impact when you can (e.g., "reduced load time by 40%" vs. "made it faster")
- Include collaborators so you capture what you did *with* others, not just what you did alone
- If an accomplishment touches multiple core values or competencies, list them all — don't be conservative
- The weekly reflection mode is great if you feel stuck; guided questions often surface wins you undervalue
- You can log accomplishments retroactively — they don't have to be fresh

## Core Values reference

Your six core values (from CoreValues.md):
1. Focus on Impact
2. Obsess over Transparency
3. Lead at Every Level
4. Think Differently
5. Take Initiative
6. Understand the Why

## Competencies reference

Your competencies framework (from Competencies_Tech.md):
- Technical Proficiency
- Communication
- Adaptability
- Problem Solving
- Impact & Influence

Each competency has a description and examples at your current level (Lead Engineer / L3) and your target level (Staff Engineer / L4).
