# HTML and Markup Stripping

tags: #nlp #text-preprocessing #cleaning #html #web-scraping
links: [[Noise Removal]] [[Encoding and Unicode]] [[Deduplication]]

---

## Definition + Intuition

**HTML and markup stripping** is the removal of structural, presentational, and metadata tags from raw web-scraped or document-sourced text, leaving only the human-readable content.

Beyond HTML, this includes: Markdown, wiki markup (MediaWiki), LaTeX, XML, BBCode, and RTF — any format where presentation instructions are embedded in the text stream alongside content.

> **Intuition**: A web page is like a gift wrapped in elaborate packaging. The HTML tags (`<div>`, `<span>`, `<nav>`) are the wrapping paper and ribbon — structurally useful for the browser but meaningless to an NLP model trying to understand the content. Stripping markup is unwrapping the gift to get to what's inside.

---

## Key Properties / Types

### HTML Structure and What to Strip / Keep

```html
<html>
  <head>
    <title>Article Title</title>         ← KEEP (informative)
    <script>...</script>                  ← STRIP (code)
    <style>...</style>                    ← STRIP (CSS)
    <meta name="description" content="..."> ← SOMETIMES KEEP
  </head>
  <body>
    <nav>Home | About | Contact</nav>     ← STRIP (navigation boilerplate)
    <article>
      <h1>Main Heading</h1>              ← KEEP (important content)
      <p>First paragraph...</p>          ← KEEP
      <a href="/link">Click here</a>      ← KEEP text, STRIP tag (optionally keep href)
      <img src="img.png" alt="A cat">    ← STRIP tag, KEEP alt text
      <table>...</table>                 ← KEEP content cells, STRIP structure
    </article>
    <footer>© 2024 Example Corp</footer> ← STRIP (boilerplate)
    <div class="ads">...</div>            ← STRIP (ads)
  </body>
</html>
```

### Approaches by Precision Level

**1. Regex stripping** (fast, imprecise):
```python
import re
text = re.sub(r'<[^>]+>', '', html)  # strip all tags
text = html.unescape(text)            # decode &amp; &lt; etc.
```
⚠️ Fails on: nested `>` in attributes, CDATA sections, malformed HTML.

**2. HTML parser** (robust, recommended):
```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(html, 'html.parser')
for tag in soup(['script', 'style', 'nav', 'footer', 'header', 'aside']):
    tag.decompose()                   # remove non-content tags
text = soup.get_text(separator=' ', strip=True)
```

**3. Content extraction libraries** (smart, task-oriented):
- **trafilatura**: extracts main article content, removes boilerplate
- **newspaper3k**: article extraction from news sites
- **jusText**: removes boilerplate by paragraph density heuristics
- **readability-lxml**: port of Firefox Reader Mode algorithm

### HTML Entities

HTML encodes special characters as named or numeric entities:

| Entity | Character | Decoded |
|--------|-----------|---------|
| `&amp;` | & | ampersand |
| `&lt;` | < | less-than |
| `&gt;` | > | greater-than |
| `&quot;` | " | quotation mark |
| `&apos;` | ' | apostrophe |
| `&nbsp;` | (non-breaking space) | space |
| `&#8217;` | ' | right single quotation |
| `&#x2019;` | ' | right single quotation (hex) |

**Always decode HTML entities** before further processing:
```python
import html
text = html.unescape(raw_html_text)
```

### Markdown Stripping

```python
import re

def strip_markdown(text):
    text = re.sub(r'#{1,6}\s+', '', text)        # headings
    text = re.sub(r'\*{1,2}(.+?)\*{1,2}', r'\1', text)  # bold/italic
    text = re.sub(r'`{1,3}.+?`{1,3}', '', text, flags=re.DOTALL)  # code
    text = re.sub(r'\[(.+?)\]\(.+?\)', r'\1', text)   # links → text
    text = re.sub(r'^\s*[-*+]\s+', '', text, flags=re.MULTILINE)  # lists
    text = re.sub(r'^\s*\d+\.\s+', '', text, flags=re.MULTILINE)  # numbered lists
    text = re.sub(r'^>{1,}\s+', '', text, flags=re.MULTILINE)     # blockquotes
    return text
```

---

## Math / Formal Notation

### Boilerplate Detection (jusText Algorithm)

JusText classifies each paragraph block $p$ as content or boilerplate using:

1. **Link density**:
$$\text{link\_density}(p) = \frac{\text{chars\_in\_links}(p)}{\text{chars}(p)}$$
Paragraphs where $\text{link\_density} > 0.33$ are likely navigation boilerplate.

2. **Word count threshold**: paragraphs with $|\text{words}(p)| < 3$ are short noise.

3. **Stop word density** (language-dependent): content paragraphs tend to have more stop words than navigation menus.

$$\text{stopword\_density}(p) = \frac{|\{w \in p : w \in \mathcal{S}\}|}{|\text{words}(p)|}$$

Paragraphs are classified as: `GOOD` (keep), `BAD` (boilerplate), `SHORT-GOOD`, `NEAR-GOOD` — using a finite state machine that considers context (paragraphs surrounded by GOOD paragraphs are more likely GOOD).

---

## Examples (Concrete)

### Before / After: news article

**Raw HTML:**
```html
<div id="header"><a href="/">Home</a> | <a href="/news">News</a></div>
<article>
  <h1>Scientists Discover New Species</h1>
  <p class="byline">By Jane Smith | March 15, 2024</p>
  <p>Researchers at MIT have <strong>discovered</strong> a new species 
  of deep-sea fish in the Pacific Ocean.</p>
  <p>The fish, named <em>Abyssalia nova</em>, was found at depths 
  exceeding 4,000 metres.</p>
</article>
<div class="ads"><script>...</script></div>
<footer>© 2024 News Corp. All rights reserved.</footer>
```

**After stripping + entity decode:**
```
Scientists Discover New Species
By Jane Smith | March 15, 2024
Researchers at MIT have discovered a new species of deep-sea fish in the Pacific Ocean.
The fish, named Abyssalia nova, was found at depths exceeding 4,000 metres.
```

### Wikipedia Markup (MediaWiki)

```
Raw:  "[[Albert Einstein|Einstein]] was born in {{birth date|1879|3|14}}."
Clean: "Einstein was born in March 14, 1879."
```

Libraries: `mwparserfromhell` for MediaWiki; `wikitextparser` for more complex templates.

---

## How It Connects to ML / NLP

| Markup Element | Why It Harms NLP | Treatment |
|---------------|-----------------|---------|
| `<script>` blocks | JavaScript syntax fills vocabulary with `{`, `}`, `var`, etc. | Always remove |
| CSS classes/IDs | `class="sidebar-promo-widget"` as tokens | Always remove |
| Navigation menus | "Home About Contact Services" repeated in every page | Remove or boilerplate-filter |
| Table structure tags | `<tr><td>` without content | Remove tags, keep cell text |
| `&nbsp;` entities | Invisible token boundaries | Decode + normalise to space |
| `alt` text on images | Can be informative content | Keep selectively |

**For LLM pretraining corpora** (Common Crawl pipeline): HTML stripping is one of the first pipeline stages — before language ID, deduplication, or quality filtering. The quality of stripping directly determines the proportion of "boilerplate" in the training data.

**Cross-links:**
- [[3.ML & DL/1.Concepts/1.Foundations/Training Data.md]] — training data quality
- [[Noise Removal]] — HTML stripping is a specific noise removal subtask

---

## Common Interview Questions

**Q: Why not just use regex to strip HTML tags?**
A: Regex `<[^>]+>` fails on: (1) `>` inside attribute values: `<div title="a > b">`; (2) multiline tags; (3) CDATA sections; (4) malformed HTML (very common on the web). A proper HTML parser like BeautifulSoup handles all of these via a state machine parser, not pattern matching. Regex is acceptable for controlled inputs; a parser is required for real-world web data.

**Q: What is the difference between HTML stripping and boilerplate removal?**
A: HTML stripping removes markup tags while keeping all text content. Boilerplate removal goes further — it identifies and removes repetitive, low-information text that happens to be plain text (navigation menus, cookie notices, legal footers, "Related articles" sections). You need both: first strip HTML, then run boilerplate detection on the resulting text.

**Q: How do you handle `<table>` elements when extracting text?**
A: Tables contain structured data — simply concatenating cells without delimiters produces confused text. Options: (1) skip tables entirely if only prose is needed; (2) convert to pipe-separated or TSV format for tasks that need table content; (3) extract caption + header row + each data row as a separate "sentence." The right choice depends on whether your downstream task needs table content (e.g. table QA needs it; sentiment analysis may not).

---

## Common Mistakes / Gotchas

- **Stripping before entity decoding**: if you strip `<p>Hello &amp; goodbye</p>` you get `Hello &amp; goodbye` — still broken. Always decode entities after stripping.
- **Removing all `<a>` tags losing anchor text**: anchor text (`<a href="...">click here</a>`) is often informative content. Remove the tag but keep the inner text.
- **Ignoring hidden `<div>` sections**: CSS `display:none` elements contain text that is never shown to users — but survives stripping. Filter by `style="display:none"` or `hidden` attribute.
- **Treating Wikipedia raw dumps as clean text**: raw Wikipedia XML contains MediaWiki markup (`[[links]]`, `{{templates}}`, `==Headings==`). The processed Wikipedia text (e.g. from HuggingFace datasets) has already been cleaned — do not clean it again.

---

## Further Reading / Paper References

- Pomikálek, J. (2011). *Removing Boilerplate and Duplicate Content from Web Corpora.* PhD thesis. — jusText
- Barbaresi, A. (2021). *Trafilatura: A Web Scraping Library and Command-Line Tool for Text Discovery and Extraction.* ACL. [[arxiv:2106.12887]]
- Wenzek, G. et al. (2020). *CCNet: Extracting High Quality Monolingual Datasets from Web Crawl Data.* [[arxiv:1911.00359]]
- BeautifulSoup docs: https://www.crummy.com/software/BeautifulSoup/
- trafilatura docs: https://trafilatura.readthedocs.io
