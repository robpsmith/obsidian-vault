---
date: 2026-02-09
type: meta
tags:
  - meta
  - guide
---

# Obsidian Vault Guide

Welcome to your Obsidian vault! This guide explains how to use your note-taking system effectively.

## 📁 Folder Structure

### 00-Templates
**Purpose**: All template files for quick note creation
**Usage**: Access via Templates plugin (Ctrl+P → "Insert template")

Contains:
- Daily Note.md - Your daily workflow foundation
- Weekly Note.md - Weekly review and planning
- Monthly Status Report.md - Professional reports for your manager
- Project Note.md - Complete project tracking
- Meeting Note.md - Structured meeting capture
- Learning Note.md - Courses, books, articles, skills
- Quick Note.md - Fast capture for fleeting thoughts

### 01-Daily
**Purpose**: Daily notes organized by year
**Structure**: `01-Daily/YYYY/YYYY-MM-DD.md`
**Usage**: Ctrl+P → "Open today's daily note" (auto-creates from template)

**Daily Workflow**:
- **Morning**: Open today's note, review yesterday, set focus
- **During day**: Log work, capture thoughts, link to projects
- **Evening**: Complete tasks, add reflections, plan tomorrow

### 02-Work
**Purpose**: All work-related content

**Subfolders**:
- **Projects/**: Individual project notes (use Project Note template)
- **Meetings/**: Meeting notes (use Meeting Note template)
- **Status Reports/**: Monthly reports for your boss

**Work Workflow**:
- Create project notes for all active work
- Link daily notes to projects you work on
- Update project status regularly
- Create monthly status reports from daily notes

### 03-Learning
**Purpose**: Organized learning materials

**Subfolders**:
- **Courses/**: Online courses, certifications, structured learning
- **Books/**: Book notes and summaries
- **Articles/**: Article highlights and takeaways
- **Skills/**: Skill development tracking (e.g., "Python Mastery")

**Learning Workflow**:
- Create learning notes for courses/books (use Learning Note template)
- Link daily notes when you study
- Track progress in learning notes
- Connect learning to work projects

### 04-Areas
**Purpose**: Ongoing areas of responsibility (non-project work)

**Examples**:
- Team management
- System maintenance
- Recurring responsibilities
- Personal development areas

### 05-Archive
**Purpose**: Completed projects and old content

**When to archive**:
- Completed projects (move from 02-Work/Projects)
- Old learning materials you've finished
- Reference material no longer actively used

### 99-Meta
**Purpose**: Vault documentation and indexes

**Contains**:
- This guide
- Tags Index
- Maps of Content (MOCs)
- System documentation

---

## 🏃 Workflows

### Daily Workflow

**Morning** (5 mins):
1. Ctrl+P → "Open today's daily note"
2. Review yesterday's note (click Previous Day link)
3. Set 1-3 focus items for today

**During Day**:
1. Log work in "Work Log" section
2. Create meeting notes, link from daily note
3. Update project notes as you work
4. Capture ideas in "Ideas & Thoughts"

**Evening** (10 mins):
1. Check off completed tasks
2. Fill in "Daily Review" section
3. Set tomorrow's priorities
4. Link any new project/learning notes created today

### Weekly Workflow

**When**: Friday afternoon or Sunday evening
**Time**: 30-45 minutes

1. Create weekly note (insert Weekly Note template)
2. Review all daily notes from the week
3. Extract key accomplishments to weekly note
4. Update project statuses
5. Process any Quick Notes into permanent notes
6. Plan next week priorities

**Search tip**: Find this week's daily notes:
```
path:01-Daily/2026 2026-02-03 OR 2026-02-04 OR ...
```

### Monthly Workflow

**When**: Last day of month or first day of new month
**Time**: 1-2 hours

1. Create Monthly Status Report (use template in 00-Templates)
2. Review all daily notes from month:
   ```
   path:01-Daily/2026 2026-02
   ```
3. Extract accomplishments from work logs
4. Update all project statuses
5. Fill in challenges, metrics, next month priorities
6. **Remove "Internal Notes" section before sending to boss**
7. Export report (see Export Strategy below)
8. Save final version in `02-Work/Status Reports/`
9. Archive any completed projects to 05-Archive

---

## 🔗 Linking Strategy

### Why Link?

Links create connections between notes, helping you:
- See how ideas relate
- Find all notes about a topic
- Build knowledge networks
- Navigate your vault efficiently

### How to Link

**Create links**: `[[Note Title]]` or `[[Note Title|Display Text]]`
**Link to headings**: `[[Note Title#Heading]]`
**Link to blocks**: `[[Note Title^block-id]]`

### What to Link

**From Daily Notes**:
- Projects you worked on: `Worked on [[Project Name]]`
- Meetings attended: `Met with team about [[Meeting: Q1 Planning]]`
- Learning done: `Studied [[Python Course]]`

**From Project Notes**:
- Related meetings: `Discussed in [[Meeting: Sprint Planning]]`
- Daily updates: Link back from daily notes
- Related projects: `Depends on [[Infrastructure Upgrade]]`

**From Learning Notes**:
- Application to work: `Used in [[Project Name]]`
- Related topics: `Related to [[Previous Course]]`
- Daily study sessions: Linked from daily notes

### Backlinks

**What are they?**: Automatic list of notes linking to current note
**Where to find**: Right sidebar → Backlinks panel
**Use case**: See all daily notes where you worked on a project

---

## 🏷️ Tagging Conventions

### Core Tags

- `#daily` - Daily notes
- `#weekly` - Weekly review notes
- `#work` - Work-related content
- `#personal` - Personal content
- `#learning` - Learning materials

### Work Sub-tags

- `#work/project` - Project notes
- `#work/meeting` - Meeting notes
- `#work/status` - Status reports

### Status Tags

- `#active` - Currently active items
- `#completed` - Finished items
- `#blocked` - Blocked/waiting
- `#review` - Needs review

### Priority Tags

- `#high-priority` - Urgent/important
- `#medium-priority` - Normal priority
- `#low-priority` - Low priority

### Learning Tags

- `#learning/course` - Structured courses
- `#learning/book` - Books
- `#learning/article` - Articles
- `#learning/skill` - Skill development

**Tip**: Use tag search in Obsidian: Click any tag to see all notes with that tag

---

## 📤 Export Strategy for Status Reports

### Internal Version (Full Details)

**Write in Obsidian**:
- Include ALL context in main sections
- Use "Internal Notes" section for private thoughts
- Add links to related notes
- Keep specific metrics and details

### Export to Boss (Clean Version)

**Method 1**: Copy from Reading View
1. Complete status report in Obsidian
2. Press Ctrl+E to switch to Reading View
3. Select sections to share (skip "Internal Notes")
4. Copy and paste into email or Word
5. Format as needed in destination

**Method 2**: PDF Export
1. Complete status report
2. Right-click note → Export to PDF
3. Edit PDF or print to clean PDF
4. Send to boss

**Method 3**: Clean in Word
1. Copy full markdown to Word
2. Remove "Internal Notes" section
3. Clean up formatting
4. Save as .docx or PDF

### Keep Both Versions

**Obsidian**: Full internal version with all details (in `02-Work/Status Reports/`)
**Sent**: Cleaned version sent to boss
**Why**: Internal version preserves context for your future reference

---

## ⌨️ Essential Keyboard Shortcuts

- **Ctrl+P**: Command palette (access all commands)
- **Ctrl+O**: Quick switcher (find notes)
- **Ctrl+N**: New note
- **Ctrl+E**: Toggle Reading/Editing view
- **Ctrl+,**: Settings
- **Ctrl+K**: Insert link
- **Ctrl+[**: Navigate back
- **Ctrl+]**: Navigate forward

**Daily Notes Plugin**:
- Open today's daily note (set in Settings → Hotkeys)

**Templates Plugin**:
- Insert template (set in Settings → Hotkeys, recommend Ctrl+T)

---

## 💡 Best Practices

### Do's

✅ **Link liberally**: Connect related notes with wikilinks
✅ **Use templates**: Consistent structure makes information easier to find
✅ **Review regularly**: Daily → Weekly → Monthly rhythm builds good habits
✅ **Tag consistently**: Follow the tagging conventions above
✅ **Archive completed work**: Keep active areas focused
✅ **Update projects frequently**: Keep status current in project notes

### Don'ts

❌ **Don't over-organize**: Use folders that exist, don't create excessive subfolders
❌ **Don't duplicate**: Link to existing notes rather than copying content
❌ **Don't ignore backlinks**: They're powerful for seeing connections
❌ **Don't skip daily reviews**: 5 minutes daily > 2 hours monthly catchup
❌ **Don't forget to process**: Quick notes should be processed regularly

### Tips for Success

**Start Simple**: Use Daily Notes template for first week, add complexity as needed
**Be Consistent**: Same time daily for notes builds habit
**Link as You Go**: Don't wait to add links, do it while writing
**Review Before Reporting**: Always review daily notes before creating status reports
**Experiment**: Adjust templates and workflows to fit your style

---

## 🚀 Getting Started Checklist

Week 1:
- [ ] Create daily note each day
- [ ] Practice linking to projects in daily notes
- [ ] Create 1-2 project notes for active work
- [ ] Take meeting notes using template

Week 2:
- [ ] Create first weekly note (Friday/Sunday)
- [ ] Review daily notes when creating weekly note
- [ ] Add tags to notes consistently
- [ ] Create learning note for something you're studying

Week 3:
- [ ] Start tracking projects more formally
- [ ] Link daily notes to projects regularly
- [ ] Practice using backlinks panel
- [ ] Process any quick notes collected

Week 4:
- [ ] Create monthly status report
- [ ] Review all daily notes from month
- [ ] Export status report to boss
- [ ] Archive any completed projects

---

## 🆘 Troubleshooting

**Daily note not creating from template?**
- Check `.obsidian/daily-notes.json` has correct template path
- Restart Obsidian
- Verify template exists in `00-Templates/`

**Templates not appearing in insert menu?**
- Check `.obsidian/templates.json` has correct folder
- Restart Obsidian
- Verify templates exist in `00-Templates/`

**Links not working?**
- Use exact note titles: `[[Exact Title]]`
- Note titles are case-sensitive
- Check for typos in link text

**Can't find a note?**
- Use Ctrl+O (Quick Switcher)
- Use search (Ctrl+Shift+F)
- Check if note is in Archive folder

---

## 📚 Additional Resources

**Obsidian Help**:
- Official docs: https://help.obsidian.md
- Community forum: https://forum.obsidian.md

**Methodologies Referenced**:
- PARA Method (Tiago Forte)
- Zettelkasten (linking and note-taking)
- GTD (Getting Things Done)

**Recommended Plugins** (to explore later):
- Dataview - Query your notes like a database
- Calendar - Visual calendar view of daily notes
- Kanban - Project management boards
- Excalidraw - Sketching and diagrams

---

**Last Updated**: 2026-02-09
**Vault Version**: 1.0
