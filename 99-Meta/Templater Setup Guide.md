---
date: 2026-02-10
type: meta
tags:
  - meta
  - setup
---

# Templater Setup Guide

Your templates now use **Templater syntax** instead of Obsidian's built-in templates. This gives you much more power, including cursor positioning and advanced date math.

## ⚙️ Installing Templater

1. Open Obsidian → Settings → Community plugins
2. Click "Browse" button
3. Search for "Templater"
4. Click "Install" on the plugin by SilentVoid00
5. Click "Enable" to activate it

## 🎯 How to Use Templates Now

### Method 1: Insert Template Command (Recommended for all templates)

**For any template (Daily, Project, Meeting, Learning, Weekly, Status Report, Quick Note):**

1. Press `Ctrl+P` (Command Palette)
2. Type "Insert template"
3. Select your desired template
4. Template inserts with:
   - ✅ Cursor positioned at first input field
   - ✅ Current date/time automatically filled
   - ✅ All Templater code executed

### Method 2: Daily Notes Plugin (For daily notes only)

The Daily Notes plugin **still works** but with limitations:
- ✅ Dates auto-fill using Templater's `2026-02-10`
- ❌ Cursor positioning (`<% tp.file.cursor() %>`) won't work
- ❌ Templater-specific features won't execute

**To create today's daily note via Daily Notes:**
- Press `Ctrl+P` → "Open today's daily note"
- Plugin auto-creates note in `01-Daily/YYYY/MMMM/YYYY-MM-DD.md`
- Templater code will execute and dates will populate

**Best practice**: Use the "Insert template" method for better cursor positioning.

## 🔧 Configure Templater (Optional)

You can optionally set hotkeys for common templates:

1. Settings → Templater
2. Scroll to "Hotkeys" section
3. Set custom hotkeys for frequently used templates
4. Example: `Ctrl+Alt+D` for Daily Note, `Ctrl+Alt+P` for Project Note

## ✨ What Your Templates Can Do Now

### Date Variables
```
2026-02-10        → 2026-02-09
Tuesday, February 10     → Monday, February 09
08:19             → 14:30
```

### Date Math
```
2026-02-11     → Tomorrow
2026-02-09    → Yesterday
2026-02-17     → 7 days from now
```

### Cursor Positioning
```
<% tp.file.cursor() %>                  → Places cursor here after insert
```

### File Operations
```
Templater Setup Guide                     → Current file title
99-Meta                  → Current folder path
```

### Other Features
```
> [!quote] I endeavour to be wise when I cannot be merry, easy when I cannot be glad, content with what cannot be mended and patient when there is no redress.
> — Elizabeth Montagu              → Random daily quote
<% tp.file.cursor() %>
             → Paste clipboard content
```

## 🚀 Using Your Templates

### Daily Note
1. `Ctrl+P` → "Insert template" → Daily Note
2. Cursor lands in "Today's Focus" section
3. Start typing your focus for the day
4. All dates auto-populated

### Weekly Note
1. `Ctrl+P` → "Insert template" → Weekly Note
2. Title shows "Week 06 - 2026" (auto-filled)
3. Daily note links auto-generated for the week

### Project Note
1. `Ctrl+P` → "Insert template" → Project Note
2. Cursor positioned at project name field
3. All dates and metadata auto-filled

### Meeting Note
1. `Ctrl+P` → "Insert template" → Meeting Note
2. Cursor ready for meeting title
3. Date, time, and current time auto-populated

### Status Report
1. `Ctrl+P` → "Insert template" → Monthly Status Report
2. Title shows current month: "February 2026 Status Report"
3. Ready for quick fill-in

### Learning Note
1. `Ctrl+P` → "Insert template" → Learning Note
2. Cursor ready for resource title
3. Date auto-populated

### Quick Note
1. `Ctrl+P` → "Insert template" → Quick Note
2. Cursor ready in idea section
3. Timestamp auto-filled

## 📋 Workflow Tips

### Setting Hotkeys for Speed

If you frequently insert daily notes, set a hotkey:

1. Settings → Hotkeys
2. Search for "Templater: Insert template"
3. Click to set a key combination
4. Suggested: `Ctrl+Alt+D` for daily, `Ctrl+Alt+P` for project

### Using Templater with Daily Notes

You can still use the Daily Notes plugin AND Templater:

- **Daily Notes plugin**: Creates the file at correct path/name
- **Templater**: Insert command gives better cursor positioning

For best results with daily notes:
- Option A: Use "Insert template" (full Templater features)
- Option B: Use Daily Notes plugin (auto-creation, simpler)

## ⚠️ Important Notes

### Mixed Syntax
Your templates use only **Templater syntax** (`<% tp.file.cursor() %>`), not Obsidian syntax (`{{ }}`).

### Dataview Still Works
Dataview queries in templates still work fine:
```dataview
List FROM "" WHERE file.cday = date(today)
```

This is separate from Templater and executes after the note is created.

### Template References
Templates in filenames:
- Use forward slashes: `00-Templates/Daily Note.md`
- Don't abbreviate or try to use aliases
- Match exact filename including capitalization

## 🆘 Troubleshooting

**Templates not inserting?**
- Ensure Templater plugin is installed and enabled
- Restart Obsidian
- Try using `Ctrl+P` → "Insert template" directly

**Dates showing as code instead of formatting?**
- Templater syntax requires `undefined` angle brackets
- Obsidian syntax uses `{{ }}`
- Check template file for syntax errors

**Cursor not positioning correctly?**
- Make sure line has `undefined`
- It should be the ONLY content on that line for best results

**"Insert template" command not appearing?**
- Templater plugin might not be enabled
- Check: Settings → Community plugins → Templater → Enable

## 📚 Learn More

- Templater docs: https://silentvoid13.github.io/Templater/
- Date format reference: https://momentjs.com/docs/#/displaying/format/
- Templater commands: Check plugin settings for full command list

---

**Templater Version**: Any recent version (install from Community plugins)
**Tested With**: Obsidian 1.x+
**Required**: Templater plugin from SilentVoid00
