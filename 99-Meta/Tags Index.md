---
date: 2026-02-09
type: meta
tags:
  - meta
  - index
---

# Tags Index

This document defines the tagging system used throughout the vault. Use tags consistently to make filtering and searching easier.

## 📋 Core Tags

### Note Types

| Tag | Usage | Examples |
|-----|-------|----------|
| `#daily` | Daily notes | All daily notes in 01-Daily |
| `#weekly` | Weekly review notes | Weekly summaries |
| `#work` | Work-related content | Projects, meetings, status reports |
| `#personal` | Personal content | Personal projects, reflections |
| `#learning` | Learning materials | Courses, books, articles, skills |
| `#meta` | Vault documentation | This guide, indexes, MOCs |

## 💼 Work Tags

### Work Categories

| Tag | Usage | Examples |
|-----|-------|----------|
| `#work/project` | Individual projects | Active work projects |
| `#work/meeting` | Meeting notes | Team meetings, one-on-ones, client calls |
| `#work/status` | Status reports | Monthly reports for management |

### Project Status

| Tag | Meaning | When to Use |
|-----|---------|-------------|
| `#active` | Currently working on | Active projects and tasks |
| `#completed` | Finished | Completed projects (before archiving) |
| `#blocked` | Waiting on something | Blocked tasks, pending decisions |
| `#on-hold` | Paused temporarily | Projects temporarily stopped |
| `#planning` | In planning phase | Not yet started execution |

### Priority

| Tag | Meaning | When to Use |
|-----|---------|-------------|
| `#high-priority` | Urgent and important | Critical deadlines, key deliverables |
| `#medium-priority` | Normal priority | Standard work items |
| `#low-priority` | Can wait | Nice-to-have items |

## 📚 Learning Tags

### Learning Types

| Tag | Usage | Examples |
|-----|-------|----------|
| `#learning/course` | Structured courses | Online courses, certifications, workshops |
| `#learning/book` | Books | Technical books, business books, self-help |
| `#learning/article` | Articles | Blog posts, papers, documentation |
| `#learning/video` | Videos | YouTube tutorials, conference talks |
| `#learning/skill` | Skill development | Language learning, tools, methodologies |

### Learning Status

| Tag | Meaning | When to Use |
|-----|---------|-------------|
| `#in-progress` | Currently learning | Active learning materials |
| `#completed` | Finished | Completed courses/books |
| `#paused` | Temporarily stopped | Will return to later |
| `#reference` | Quick reference | Materials to reference, not study sequentially |

## 🎯 Context Tags

### Timeframes

| Tag | Usage |
|-----|-------|
| `#q1`, `#q2`, `#q3`, `#q4` | Quarterly goals and projects |
| `#2026` | Year-specific items |

### Areas of Focus

Create custom tags for your specific areas:

**Example Technical Tags**:
- `#python`
- `#javascript`
- `#devops`
- `#architecture`

**Example Domain Tags**:
- `#backend`
- `#frontend`
- `#database`
- `#security`

**Example Business Tags**:
- `#strategy`
- `#leadership`
- `#team-management`

## 🔍 Special Purpose Tags

| Tag | Purpose | Usage |
|-----|---------|-------|
| `#quick` | Quick capture notes | Fleeting thoughts to process later |
| `#process-later` | Needs processing | Notes that need to be expanded or filed |
| `#review` | Needs review | Content that needs checking or updating |
| `#question` | Open questions | Unresolved questions or research needed |
| `#idea` | Ideas to explore | Ideas for projects, features, content |
| `#decision` | Key decisions | Important decisions for reference |

## 📖 Tag Usage Guidelines

### Combining Tags

Tags work best in combination. Examples:

```markdown
#work/project #active #high-priority #python
```
This is an active, high-priority work project related to Python.

```markdown
#learning/course #in-progress #javascript #frontend
```
An in-progress JavaScript course focused on frontend development.

```markdown
#work/meeting #q1 #team-management
```
A Q1 meeting about team management.

### How Many Tags?

**Minimum**: 1-2 core tags (e.g., `#work`, `#daily`)
**Optimal**: 3-5 tags providing useful context
**Maximum**: Avoid going over 7-8 tags

**Too few**: `#work`
**Just right**: `#work/project #active #python #backend`
**Too many**: `#work #project #active #important #python #code #backend #api #database #2026 #q1`

### Tag Hierarchy

Use forward slashes for hierarchies:

```
#work
  #work/project
  #work/meeting
  #work/status

#learning
  #learning/course
  #learning/book
  #learning/article
```

**Search tip**: Searching `#work` finds all work tags including sub-tags.

## 🛠️ Maintenance

### Regular Reviews

**Monthly**: Review tags in use
- Are you using tags consistently?
- Are there tags you never search for? (Remove them)
- Do you need new tags for emerging topics?

**Quarterly**: Update this index
- Add new tags you've created
- Document any tag convention changes
- Archive unused tag definitions

### Tag Cleanup

**Signs you need cleanup**:
- Similar tags with slightly different names (`#mtg` vs `#meeting`)
- Tags used only once or twice
- Inconsistent capitalization
- Vague tags that don't help filtering

**How to clean up**:
1. Search for tag: `tag:#old-tag`
2. Find all notes with that tag
3. Replace with standardized tag
4. Update this index

## 📊 Useful Tag Searches

Find specific content using tag searches:

**All active work projects**:
```
tag:#work/project tag:#active
```

**High priority items across all areas**:
```
tag:#high-priority
```

**In-progress learning materials**:
```
tag:#learning tag:#in-progress
```

**All work from Q1 2026**:
```
tag:#work tag:#q1 tag:#2026
```

**Meeting notes from a specific month**:
```
tag:#work/meeting path:01-Daily/2026 2026-02
```

## 🎨 Tag Best Practices

### Do's

✅ **Be consistent**: Use the same tag for the same concept
✅ **Use hierarchies**: `#work/project` is better than `#workproject`
✅ **Tag on creation**: Add tags when creating note, not later
✅ **Review tag cloud**: Use Obsidian's tag pane to see all tags
✅ **Document new tags**: Add to this index when creating new conventions

### Don'ts

❌ **Don't over-tag**: More tags ≠ better organization
❌ **Don't create synonyms**: Pick one tag name and stick with it
❌ **Don't use spaces**: Use `#high-priority` not `#high priority`
❌ **Don't capitalize randomly**: Decide on convention (`#Python` or `#python`)
❌ **Don't tag everything**: Some notes don't need many tags

## 🔄 Evolution

This tagging system will evolve with your needs:

**Week 1-4**: Use core tags only, learn the system
**Month 2-3**: Add subject-specific tags for your domain
**Month 3+**: Refine based on what you actually search for

**Remember**: Tags serve you. If a tag doesn't help you find things, don't use it.

---

**Last Updated**: 2026-02-09
**Version**: 1.0
