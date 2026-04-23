# Creditsafe Report Analyzer — Setup

## Prerequisites

### pdftotext (required)
Install poppler to get the `pdftotext` CLI:

```bash
# macOS
brew install poppler

# Ubuntu/Debian
sudo apt-get install poppler-utils
```

Verify installation
```bash
which pdftotext  # Should return a path
pdftotext -v     # Should show version info
```

## Access Requirements

- Local filesystem access to Creditsafe PDF files
- No API keys or network access required for basic analysis
