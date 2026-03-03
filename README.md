# chatgpt-to-claude

Convert a ChatGPT export into a fully preserved local archive, with Claude filling in what OpenAI's export omits.

## What it does

For every conversation in your ChatGPT export ZIP:

| Content | Strategy |
|---|---|
| Conversation text | ✅ Fully preserved |
| User-uploaded files | 📦 Extracted from ZIP if present; otherwise permanently unrecoverable (OpenAI blocks API retrieval of `user_data` files by design) |
| DALL·E generated images | 📦 Extracted from ZIP by size-match → ☁️ OpenAI Files API → 🤖 Recreated by Claude using original prompt + seed |
| ChatGPT file outputs (code, CSV, docs) | 🤖 Recreated by Claude from conversation context |
| Unrecoverable assets | ❌ Clearly documented inline with all available metadata |

**Recreated artifacts** are marked with a prominent banner in the Markdown so you always know what's original vs. regenerated.

**Outputs per conversation:**
- `<conversation>.md` — full chat history with inline images and file links
- `assets/` — all recovered or recreated binary files

**Global outputs:**
- `INDEX.md` — master index with summaries, asset stats, and links to every conversation
- `manifest.json` — deduplication log; re-running never reprocesses completed conversations

## Requirements

- Python 3.9+
- [Anthropic API key](https://console.anthropic.com/) (required — used for summaries, recreation, and optional upload)
- [OpenAI API key](https://platform.openai.com/) (optional — improves DALL·E image recovery via Files API)

## Install

```bash
pip3 install anthropic openai
```

## Usage

### 1. Export your ChatGPT history

1. ChatGPT → **Settings → Data Controls → Export Data**
2. Wait for the email, download the ZIP

### 2. Run

```bash
python3 chatgpt_to_claude.py path/to/export.zip
```

The script will interactively prompt for API keys (or read from environment variables):

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...
python3 chatgpt_to_claude.py path/to/export.zip
```

### 3. Review, then optionally upload to Claude

After processing, the script asks whether to upload each conversation to the Claude API. Uploaded conversations get a reference ID stored in `manifest.json`.

## Output structure

```
/tmp/chatgpt-to-claude/
├── INDEX.md                          ← master index + summaries
├── manifest.json                     ← deduplication + upload log
├── 0001_My_First_Chat/
│   ├── 0001_My_First_Chat.md         ← full conversation
│   └── assets/
│       ├── file-abc123.webp          ← recovered DALL·E image
│       └── file-xyz789_RECREATED.md  ← Claude-recreated artifact (marked)
├── 0002_Another_Topic/
│   └── ...
```

## Resuming conversations in Claude

To pick up any conversation with full context, upload the relevant `.md` file at the start of a Claude chat. Claude will have the complete history and can continue from where you left off.

## Re-running

Running the script again on the same or an updated ZIP is safe — `manifest.json` tracks every processed conversation and skips it on subsequent runs.

## Known limitations

- **User-uploaded files**: OpenAI's `user_data` purpose files are [explicitly blocked from API download](https://community.openai.com/t/file-uploads-error-why-can-t-i-download-files-with-purpose-user-data/1357013). If they're not in the ZIP, they're gone.
- **DALL·E images**: Not included in the ChatGPT export ZIP. The script attempts OpenAI Files API recovery, then falls back to Claude generating a detailed description + regeneration prompt.
- **Voice / video**: Audio from Advanced Voice Mode is not exported by OpenAI.
- **Team / Enterprise**: Workspace conversations cannot be exported at all (OpenAI restriction).

## License

MIT
