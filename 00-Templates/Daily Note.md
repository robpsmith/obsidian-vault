---
date: <% tp.date.now("YYYY-MM-DD") %>
type: daily
tags:
  - daily
---

# <% tp.date.now("dddd, MMMM DD, YYYY") %>

## 📋 Today's Focus

- [ ] <% tp.file.cursor() %>

## 💼 Work Log

### Completed Tasks

-

### Notes & Decisions

## 📝 Meetings

-

## 💡 Ideas & Thoughts

---
## ✅ Daily Review

### Tomorrow's priorities

1.
2.
3.

## Links

### Notes created today
```dataview
List FROM "" WHERE file.cday = date(today) SORT file.ctime asc
```

### Notes last touched today
```dataview
List FROM "" WHERE file.mday = date(today) SORT file.mtime asc
```

**Previous**: [[<% tp.date.now("YYYY/MMMM/YYYY-MM-DD", -1) %>]] | **Next**: [[<% tp.date.now("YYYY/MMMM/YYYY-MM-DD", 1) %>]]