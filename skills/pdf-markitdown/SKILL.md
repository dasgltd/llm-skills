---
name: pdf-markitdown
description: Uses Microsoft's MarkItDown library to extract text and convert PDF files into Markdown. Trigger this skill whenever handling, reading, or parsing PDF files.
---

# PDF MarkItDown

Use the `markitdown` package (by Microsoft) whenever you need to read, extract text, or convert PDF files. This library optimizes the extraction for Large Language Models, preserving tables, headings, and structure.

## Setup Instructions

MarkItDown requires **Python 3.9 or higher**. 
If not already installed, create a virtual environment with an appropriate Python version (e.g., `/opt/homebrew/bin/python3.14`) and install it:

```bash
python3 -m venv venv
source venv/bin/activate
pip install "markitdown[all]"
```
*(Note: Using `markitdown[all]` or `markitdown[pdf]` is mandatory to get the necessary dependencies for PDF parsing).*

## How to use

### 1. Via Python Script
```python
from markitdown import MarkItDown

md = MarkItDown()
result = md.convert("caminho/para/o/arquivo.pdf")

# Para salvar em um arquivo .md:
with open("saida.md", "w", encoding="utf-8") as f:
    f.write(result.text_content)
```

### 2. Via CLI
```bash
markitdown arquivo.pdf > saida.md
```

## Best Practices
- When analyzing large numbers of PDFs, write a short Python loop to process them all and dump them into a dedicated directory.
- Treat the extracted markdown as the source of truth for RAG or text analysis tasks.
