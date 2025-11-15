# CLAUDE.md - Douyin Video Capture Assistant

## Project Overview

**Project Name:** Douyin Video Capture Assistant (抖音视频采集助手)
**Version:** 5.0
**Type:** Single-Page Web Application
**Primary Language:** HTML/JavaScript (Vanilla JS)
**Purpose:** A web-based tool for capturing and analyzing Douyin (Chinese TikTok) video metadata through N8N workflow integration

### What This Application Does

This is a lightweight browser-based tool that:
- Extracts Douyin video links from share text
- Fetches comprehensive video metadata via N8N webhook API
- Displays video information including stats, author details, tags, and categories
- Maintains local history of captured videos
- Exports history data to CSV format
- Supports video download functionality

---

## Repository Structure

```
douyin-capture/
├── index.html          # Main application file (all-in-one)
├── README.md           # Project readme (UTF-16 encoded, minimal)
└── CLAUDE.md          # This file - AI assistant guidelines
```

### Architecture Pattern

**Single-File Application:** This project uses a monolithic architecture where all HTML, CSS, and JavaScript code resides in a single `index.html` file. This design choice offers:
- Zero build process requirement
- Easy deployment (drag-and-drop to any web server)
- No dependencies or package management
- Instant loading and execution

---

## Key Components and Responsibilities

### 1. User Interface (`index.html` lines 1-223)

**CSS Variables (lines 8-23):**
- Implements automatic dark mode using `prefers-color-scheme`
- Custom properties for theming: `--bg`, `--text`, `--card`, `--accent`, `--shadow`

**Main Sections:**
- Input field for Douyin share text/URL
- Action buttons (Capture + Download)
- Status/loading indicator
- Results display card with rich metadata
- History panel with export functionality

### 2. Core JavaScript Functions (`index.html` lines 240-600)

#### Configuration
```javascript
const WEBHOOK_URL = 'https://n8ntext.chaim.top/webhook/ca0a5e3b-e92a-4ed4-8a4f-31bac17c5424'
```
**Location:** Line 242
**Critical:** This URL points to the N8N workflow endpoint

#### Primary Functions

**`extractDouyinUrl(text)` (lines 307-325)**
- Purpose: Intelligent URL extraction from Chinese share text
- Supports multiple URL formats:
  - Short links: `https://v.douyin.com/xxxxxxx/`
  - Full links: `https://www.douyin.com/video/[id]`
  - Mobile links: `https://m.douyin.com/video/[id]`
- Returns: Extracted URL string or `null`

**`fetchData()` (lines 330-491)**
- Purpose: Main data collection function
- Flow:
  1. Validates input and extracts URL
  2. Sends POST request to N8N webhook with payload
  3. Processes response (handles array or object format)
  4. Renders video card with metadata
  5. Saves to localStorage history
- Error Handling: Comprehensive CORS, network, and HTTP status error detection

**`saveHistory(data)` (lines 248-256)**
- Stores up to 10 most recent captures in localStorage
- Key: `douyinHistory`
- Adds timestamp to each entry

**`renderHistory()` (lines 261-276)**
- Renders history list from localStorage
- Shows: title, author, likes, timestamp

**`exportCSV()` (lines 281-300)**
- Exports history to CSV with UTF-8 BOM
- Columns: 标题, 作者, 点赞数, 视频链接, 采集时间
- Filename: `douyin_history_YYYY-MM-DD.csv`

**`downloadVideo()` (lines 530-585)**
- Downloads video using direct link
- Validates currentVideoData exists
- Creates temporary `<a>` element with download attribute
- Filename: Truncated title + `.mp4`

**`copyLink(text)` (lines 497-522)**
- Clipboard API integration
- Shows temporary toast notification on success

### 3. Data Flow

```
User Input → extractDouyinUrl() → fetchData() → N8N Webhook → Process Response
                                                                       ↓
                                                           Render Card + Save History
```

---

## N8N Integration Details

### Request Format (lines 355-360)

```javascript
{
  "text": "完整的分享文本",
  "url": "提取的链接",
  "timestamp": "2025-11-15T12:00:00.000Z",
  "source": "douyin-capture-tool"
}
```

### Expected Response Format

The application handles two response formats:

**Array Format:**
```javascript
[
  {
    "视频描述": "string",
    "作者信息": {
      "作者昵称": "string",
      "抖音号": "string"
    },
    "视频数据": {
      "点赞数": number,
      "评论数": number,
      "分享数": number,
      "收藏数": number
    },
    "视频时长": "string",
    "发布时间": "string",
    "资源链接": {
      "视频直链": "string"
    },
    "视频ID": "string",
    "话题标签": [
      { "话题名称": "string" }
    ],
    "内容分类": {
      "完整分类路径": "string"
    }
  }
]
```

**Object Format:**
```javascript
{
  "title": "string",
  "author": "string",
  "likes": number,
  // ... simplified fields
}
```

### Response Processing (lines 389-406)
The code implements fallback logic to handle both Chinese and English field names.

---

## Development Guidelines for AI Assistants

### 1. Code Modification Principles

**DO:**
- Maintain the single-file architecture
- Preserve Chinese language UI elements and comments
- Keep CSS-in-JS styling approach
- Maintain localStorage persistence logic
- Test with actual Douyin URLs before committing

**DON'T:**
- Split the application into multiple files
- Add build tools or dependencies (webpack, npm, etc.)
- Change the N8N webhook URL without user confirmation
- Modify the N8N request/response contract without testing
- Remove error handling or logging

### 2. Coding Conventions

**JavaScript Style:**
- Use ES6+ features (async/await, arrow functions, template literals)
- Prefer `const` over `let`, avoid `var`
- Use destructuring for object properties
- JSDoc comments for all major functions

**Naming Conventions:**
- Functions: camelCase with descriptive names
- Variables: camelCase
- Constants: UPPER_SNAKE_CASE (e.g., `WEBHOOK_URL`)
- CSS classes: kebab-case

**Comments:**
- Chinese comments for UI-related code
- English comments for technical/algorithmic code
- JSDoc format for function documentation

### 3. Testing Checklist

Before committing changes, verify:
- [ ] Dark mode works correctly
- [ ] URL extraction handles all three URL formats
- [ ] Error messages are user-friendly and in Chinese
- [ ] localStorage operations don't throw errors
- [ ] CSV export includes UTF-8 BOM for Excel compatibility
- [ ] Video download fallback works if direct download fails
- [ ] Mobile responsive layout is preserved

### 4. Common Modification Scenarios

#### Adding New Video Metadata Fields

**Location:** Lines 389-438
**Steps:**
1. Extract field from response (line 389-406)
2. Add to `window.currentVideoData` (line 441-454)
3. Update display card HTML (line 416-438)
4. Add to `saveHistory()` data (line 248-256)
5. Update CSV export columns (line 285-292)

#### Changing N8N Endpoint

**Location:** Line 242
**Required Actions:**
1. Update `WEBHOOK_URL` constant
2. Verify payload format matches new endpoint expectations
3. Test CORS configuration
4. Update response parsing if format changed

#### Modifying Error Handling

**Location:** Lines 459-490
**Pattern:**
```javascript
catch (err) {
  // 1. Log to console
  console.error('❌ 请求失败:', err);

  // 2. Determine error type
  let errorMessage = '网络连接失败';
  let suggestion = '请检查...';

  // 3. Check error conditions
  if (err.name === 'TypeError' && err.message.includes('CORS')) {
    // Handle CORS
  }

  // 4. Display user-friendly message
  statusDiv.innerHTML = `<div class="error">...</div>`;
}
```

### 5. Performance Considerations

**Current Optimizations:**
- No external dependencies (zero network overhead)
- CSS animations use `transform` (GPU-accelerated)
- History limited to 10 items (prevents localStorage bloat)
- Debounced UI updates

**Potential Bottlenecks:**
- Large CSV exports (>1000 items) may freeze UI
- N8N webhook response time varies (3-10 seconds typical)

### 6. Security Notes

**Data Storage:**
- All history stored in browser localStorage (client-side only)
- No server-side persistence
- User data never leaves their device except during N8N API calls

**N8N Communication:**
- Uses HTTPS endpoint
- CORS must be enabled on N8N workflow
- No authentication tokens (webhook is public)
- Rate limiting should be implemented on N8N side

**XSS Prevention:**
- User input is URL-extracted only (reduces injection surface)
- Template literals auto-escape in most contexts
- Consider sanitizing `title` and `author` fields if displaying raw HTML

### 7. Browser Compatibility

**Required APIs:**
- `fetch()` - Modern browsers (IE11+ with polyfill)
- `async/await` - ES2017+
- `localStorage` - Universal support
- `navigator.clipboard` - HTTPS required, fallback needed for HTTP
- CSS Custom Properties - IE11 unsupported (dark mode won't work)
- CSS Grid - Modern browsers only

**Target Browsers:**
- Chrome/Edge 88+
- Firefox 78+
- Safari 14+
- Mobile browsers (iOS Safari 14+, Chrome Mobile)

### 8. Deployment

**Static Hosting Options:**
- GitHub Pages (recommended for this repo)
- Netlify Drop
- Vercel
- Any web server (nginx, Apache)

**Deployment Steps:**
1. Ensure `index.html` is in repository root
2. Configure HTTPS (required for clipboard API)
3. Verify N8N webhook allows requests from hosting domain (CORS)
4. Test on production URL before announcing

**GitHub Pages Specific:**
```bash
# Enable GitHub Pages in repo settings
# Source: Deploy from branch
# Branch: claude/claude-md-mi08l1lmhti0ye4a-013s7eeNfAXwShmiyb2QSeqK or main
# Folder: / (root)
```

### 9. Troubleshooting Guide

**Issue: "CORS跨域问题"**
- **Cause:** N8N workflow not allowing origin
- **Fix:** Add CORS headers in N8N webhook response node
- **Code location:** Lines 466-467

**Issue: "未检测到有效的抖音链接"**
- **Cause:** URL format not matching regex patterns
- **Fix:** Update `extractDouyinUrl()` patterns (lines 309-316)
- **Test:** Add console.log to see actual input

**Issue: CSV export shows garbled Chinese**
- **Cause:** Missing UTF-8 BOM
- **Fix:** Already implemented at line 294 (`\ufeff`)
- **Verify:** Check Excel import settings

**Issue: Download button doesn't work**
- **Cause:** `window.currentVideoData` not set
- **Fix:** Ensure `fetchData()` completed successfully
- **Code location:** Lines 535-543

### 10. Version History Context

**v5.0 (Current)** - Commit 79f7787
- Updated index.html with enhanced features
- Added download functionality
- Improved error handling

**v1.0 (Initial)** - Commit 5bc2db1
- 初始版本 (Initial version)
- Basic capture functionality

---

## Git Workflow

### Branch Strategy

**Main Branch:** Not explicitly defined in current state
**Current Working Branch:** `claude/claude-md-mi08l1lmhti0ye4a-013s7eeNfAXwShmiyb2QSeqK`

### Commit Message Conventions

**Observed Pattern:**
- Use descriptive messages in English or Chinese
- Reference specific changes (e.g., "Update index.html")

**Recommended Format:**
```
<type>: <description>

[optional body]

Examples:
feat: Add video thumbnail preview
fix: Resolve CORS error handling
docs: Update CLAUDE.md with API changes
style: Improve dark mode contrast
refactor: Extract URL parsing logic
```

### Pre-commit Checklist

1. Test in both light and dark mode
2. Verify N8N integration works
3. Check browser console for errors
4. Test mobile responsive layout
5. Validate CSV export
6. Ensure Chinese characters display correctly

---

## Future Enhancement Ideas

### Potential Features

1. **Video Thumbnail Preview**
   - Add cover image from N8N response
   - Location: Insert in `.card` div after title

2. **Batch Processing**
   - Support multiple URLs at once
   - Queue system for API requests

3. **Advanced Filtering**
   - Search history by author/title
   - Filter by date range

4. **Analytics Dashboard**
   - Aggregate stats across all captured videos
   - Charts using Chart.js or similar

5. **Browser Extension**
   - Right-click context menu on Douyin links
   - Auto-capture from Douyin website

6. **PWA Support**
   - Add manifest.json
   - Service worker for offline access
   - Install as app on mobile

### Technical Improvements

1. **Error Recovery**
   - Retry logic for failed requests
   - Exponential backoff

2. **Input Validation**
   - More robust URL pattern matching
   - Validate N8N response schema

3. **Accessibility**
   - ARIA labels
   - Keyboard navigation
   - Screen reader support

4. **Performance**
   - Lazy load history items
   - Virtual scrolling for large lists
   - IndexedDB for large datasets

---

## Quick Reference

### File Locations

| Component | Line Range | Purpose |
|-----------|------------|---------|
| CSS Variables | 8-23 | Theme configuration |
| Webhook URL | 242 | N8N endpoint |
| URL Extraction | 307-325 | Douyin link parsing |
| Main Fetch | 330-491 | API integration |
| History Management | 248-276 | localStorage ops |
| CSV Export | 281-300 | Data export |
| Download Function | 530-585 | Video download |
| Error Handling | 459-490 | User feedback |

### Key localStorage Keys

- `douyinHistory` - Array of video capture records (max 10)

### External Dependencies

**None** - This is a dependency-free application

### API Endpoints

- N8N Webhook: `https://n8ntext.chaim.top/webhook/ca0a5e3b-e92a-4ed4-8a4f-31bac17c5424`

---

## Working with This Codebase as an AI Assistant

### Before Making Changes

1. **Read the entire `index.html` file** - It's only 604 lines
2. **Understand the N8N contract** - Request and response formats
3. **Check console logs** - Extensive logging already in place
4. **Ask for N8N access** - If endpoint changes are needed

### When Adding Features

1. **Use the TodoWrite tool** to track multi-step changes
2. **Preserve existing functionality** - This is production code
3. **Match the coding style** - Follow existing patterns
4. **Test thoroughly** - No automated tests exist
5. **Update this CLAUDE.md** - Document new features

### When Fixing Bugs

1. **Reproduce first** - Understand the exact error
2. **Check error handling** - May just need better messages
3. **Test edge cases** - Empty inputs, malformed URLs, network failures
4. **Preserve user data** - Don't break localStorage format

### When Refactoring

1. **Get user approval** - Don't surprise them with structural changes
2. **Keep it simple** - Complexity is the enemy of maintainability
3. **Maintain single-file architecture** - It's a feature, not a bug
4. **Test in multiple browsers** - Vanilla JS compatibility varies

---

## Contact and Support

**Repository:** chaim709/douyin-capture
**Current Branch:** claude/claude-md-mi08l1lmhti0ye4a-013s7eeNfAXwShmiyb2QSeqK

For questions about this codebase, refer to:
1. This CLAUDE.md file
2. Inline code comments in index.html
3. Git commit history for change context

---

**Last Updated:** 2025-11-15
**Document Version:** 1.0
**Maintained by:** AI Assistants working on this repository
