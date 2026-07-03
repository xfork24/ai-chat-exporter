# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Tampermonkey userscript** that exports AI chat conversations from multiple platforms into Markdown or JSON format. It is a single-file JavaScript project with no build system or npm dependencies.

**Supported platforms**: ChatGPT, Claude, Copilot, Gemini, Grok

## Development Commands

Since this is a userscript (not a Node.js project), there are no npm scripts. Development workflow:

- **Edit the script**: Modify `ai-chat-exporter.user.js` directly
- **Test locally**: Install the script in Tampermonkey and open a chat on any supported platform
- **Test with reference DOM**: HTML samples in `reference-html-dom/` can be loaded to verify selectors without live API access
- **Test DOM selectors**: Use `test-ai-chat-exporter.user.js` as a separate Tampermonkey script to verify DOM selectors are stable
- **Version bump**: Use Commitizen (`cz commit` or `cz bump`) — configured in `.cz.toml` for semver versioning

## Architecture

### Platform Detection
Platform is detected via hostname at line 367-384:
- `CHATGPT_HOSTNAMES`: chat.openai.com, chatgpt.com
- `GEMINI_HOSTNAMES`: gemini.google.com
- `CLAUDE_HOSTNAMES`: claude.ai
- `COPILOT_HOSTNAMES`: www.copilot.com
- `GROK_HOSTNAMES`: grok.com

### Core Components

1. **Platform Selectors** (lines 327-364): Each platform has dedicated constants for DOM selectors:
   - `CHATGPT_*` constants for ChatGPT-specific selectors
   - `GEMINI_*` constants for Gemini-specific selectors
   - etc.

2. **TurndownService** (inlined, starting line 404): Customized HTML-to-Markdown converter with platform-specific rules added at lines 1211-1700+:
   - `chatgptRemoveReactions`, `chatgptRemoveH6ChatGPTSaid`
   - `copilotRemoveReactions`, `copilotCodeBlock`, `copilotFooterLinks`
   - `claudeCodeBlock`
   - `grokPreserveNewlines`, `grokCodeBlock`
   - `geminiCodeLanguageLabel`

3. **Export Functions**:
   - `getMarkdownExport()` - Generates Markdown with YAML front matter, TOC, and formatted messages
   - `getJSONExport()` - Generates structured JSON output
   - `downloadFile()` - Triggers file download

4. **UI Components**:
   - Export controls container (floating buttons)
   - Chat outline panel (collapsible, shows message structure)
   - Settings via Tampermonkey menu commands

### Key Constants
- `EXPORTER_VERSION` (line 27): Current version
- `CURRENT_PLATFORM` (line 367): Detected platform identifier
- `OUTPUT_FILE_FORMAT_DEFAULT` (line 38): Default filename template

### Message Extraction Flow
1. Detect platform via hostname
2. Use platform-specific selectors to find message containers
3. Extract user queries and AI responses
4. Apply Turndown rules to convert HTML to Markdown
5. Generate YAML front matter with metadata
6. Build TOC linking to each message pair
7. Return Markdown or JSON output

## File Structure

```
ai-chat-exporter.user.js       # Main userscript (2973 lines)
test-ai-chat-exporter.user.js  # DOM selector tester utility
reference-html-dom/            # Sample HTML DOM files for offline testing
sample-exports/               # Example Markdown/JSON exports
regex.md                       # Regex search guide
tools/set-gemini-chat-title-prefix.user.js  # Companion script
```

## Important Notes

- **No TypeScript**: Pure JavaScript userscript (not TypeScript)
- **No build step**: Edit and reload directly in Tampermonkey
- **DOM-dependent**: Selectors may break when platforms update their UI — check `reference-html-dom/` for historical DOM samples
- **Turndown inlined**: The Turndown library is embedded (not imported) to keep the script self-contained
- **GM_* APIs**: Uses Tampermonkey APIs (`GM_getValue`, `GM_setValue`, `GM_registerMenuCommand`) for persistent settings
