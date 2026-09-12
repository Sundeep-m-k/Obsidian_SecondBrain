# Tokenization for CJK Languages

tags: #nlp #tokenization #chinese #japanese #korean #cjk #multilingual
links: [[SentencePiece]] [[Tokenization — BPE]] [[WordPiece and Unigram LM]] [[Agglutinative Languages]] [[Morphological Typology]]

---

## Definition + Intuition

**CJK** = Chinese, Japanese, Korean — three writing systems that present unique challenges for tokenization because they do not use spaces to delimit words (Chinese and Japanese) or have a complex morphological system with spaces that may not align with semantic units (Korean).

> **Intuition**: In English, "New York" is two clearly delimited tokens. In Chinese, 纽约 (Niǔyuē) is two characters with no space — and the question is whether to treat it as two characters or one word-token. Chinese text is an unbroken stream of characters; word segmentation is a non-trivial NLP task in itself, not just a preprocessing step.

---

## Key Properties / Types

### Chinese Word Segmentation (CWS)

Chinese has no whitespace between words. The fundamental question: **what is a word?**

```
"我爱自然语言处理"
= 我 爱 自然 语言 处理
= I  love  natural  language processing
= "I love natural language processing"

Possible segmentations:
  Character: 我 | 爱 | 自 | 然 | 语 | 言 | 处 | 理   (8 chars)
  Word:       我 | 爱 | 自然 | 语言 | 处理             (5 words)
  Phrase:     我 | 爱 | 自然语言处理                    (3 units)
```

**Approaches:**
1. **Character-level**: treat each Chinese character as a token. Simple, no ambiguity. Used by many Chinese BERT models (Chinese-BERT uses character tokenization).
2. **Word-level**: use a word segmentation system (Jieba, PKU segmenter) to find word boundaries, then tokenize.
3. **Subword (BPE/SentencePiece)**: learn subword units from raw character sequences. Naturally finds frequent multi-character units.

**The character-level default for Chinese**: since Chinese characters carry strong semantic content (unlike Latin characters), character-level tokenization works surprisingly well. Chinese-BERT (Cui et al.) is character-level and achieves state-of-the-art on most Chinese NLP tasks.

### Japanese Tokenization

Japanese uses three writing systems:
- **Hiragana**: phonetic syllabary (phonological grammar markers)
- **Katakana**: phonetic syllabary (loanwords, emphasis)
- **Kanji**: Chinese-derived logographic characters

Plus spaces are not used between words, and sentences can mix all three scripts in one clause.

**Japanese tokenizers:**
- **MeCab**: fast morphological analyzer; uses IPADIC/UniDic dictionaries
- **Sudachi**: modern tokenizer with three granularity levels (short/middle/long)
- **spaCy Japanese**: wraps SudachiPy

```python
import MeCab
m = MeCab.Tagger()
print(m.parse("東京都に住んでいます"))
# 東京  名詞,固有名詞,地域,一般,*,*,東京,トウキョウ,トウキョウ
# 都    名詞,接尾,地域,*,*,*,都,ト,ト
# に    助詞,格助詞,一般,*,*,*,に,ニ,ニ
# 住ん  動詞,自立,*,*,五段・マ行,連用タ接続,住む,スン,スン
# で    助動詞,*,*,*,特殊・デ,基本形,で,デ,デ
# い    動詞,非自立,*,*,一段,連用形,いる,イ,イ
# ます  助動詞,*,*,*,特殊・マス,基本形,ます,マス,マス
```

### Korean Tokenization

Korean is written with spaces but the spacing is between **eojeols** (spacing units) which are morphological clumps — not individual morphemes. Korean is agglutinative, so each eojeol may contain a noun + case marker + verb + tense marker etc.

```
"나는 학교에 갑니다"
= 나 + 는  학교 + 에  가 + ㅂ니다
= I + TOP school + LOC go + FORMAL.PRES

Eojeol level: "나는" | "학교에" | "갑니다"  (3 tokens — spaces)
Morpheme level: 나 | 는 | 학교 | 에 | 가 | ㅂ니다  (6 morphemes — correct for NLP)
```

**Korean tokenizers**: KoNLPy (wraps Kkma, Hannanum, Twitter/Okt), Mecab-ko.

---

## Math / Formal Notation

### Chinese Word Segmentation as Sequence Labeling

Frame CWS as a character-level sequence labeling problem:

Each character $c_i$ gets a label in $\{B, M, E, S\}$:
- **B**: Beginning of a multi-char word
- **M**: Middle of a multi-char word
- **E**: End of a multi-char word
- **S**: Single-character word

$$P(y_1, \ldots, y_n \mid c_1, \ldots, c_n) = \prod_{i=1}^{n} P(y_i \mid y_1^{i-1}, c_1^n)$$

Modelled with BiLSTM-CRF or BERT + CRF.

**Example:**
```
自 然 语 言 处 理
B  E  B  E  B  E
= 自然 (natural) | 语言 (language) | 处理 (processing)
```

### Maximum Matching (Jieba)

A simple but effective baseline — greedy forward maximum matching:

```
Dictionary: {自然, 自然语言, 语言, 语言处理, 处理}

Input: 自然语言处理

1. Try longest match starting at 自: 自然语言 ✓ (length 4)
2. Try longest match starting at 处: 处理 ✓ (length 2)
Result: 自然语言 | 处理
```

**Correct answer**: 自然语言处理 should be split as 自然 + 语言处理 or 自然语言 + 处理 depending on context — this illustrates segmentation ambiguity.

---

## Examples (Concrete)

**Chinese character vs word tokenization:**
```
Sentence: 我在北京工作
= I am-working-in Beijing

Character: 我 | 在 | 北 | 京 | 工 | 作   (6 tokens)
Word:       我 | 在 | 北京 | 工作          (4 tokens)

For NER: "北京" (Beijing) is one entity — word-level is better
For Chinese-BERT: character-level works well because BERT attends across chars
```

**Japanese mixed-script sentence:**
```
私はNLPの研究者です。(Watashi wa NLP no kenkyuusha desu)
= I am an NLP researcher.

Script: kanji(私) hiragana(は) latin(NLP) hiragana(の) kanji(研究者) hiragana(です) punct(。)
MeCab: 私 | は | NLP | の | 研究 | 者 | です | 。
```

---

## How It Connects to ML / NLP

| Language | Challenge | Solution |
|----------|-----------|---------|
| Chinese | No spaces; ambiguous segmentation | Character-level (BERT) or Jieba + BPE |
| Japanese | Three scripts mixed; morphological complexity | MeCab / SudachiPy + BPE |
| Korean | Spaces between eojeols; morphologically complex | KoNLPy morpheme tokenizer + BPE |
| All CJK | Character_coverage must be 1.0 in SentencePiece | `character_coverage=1.0` |

**mBERT Chinese tokenization**: character-level with no word segmentation. Each Chinese character becomes one token (with `##` prefix if continuation in a word — though mBERT doesn't do word segmentation). Works well in practice.

**Cross-links:**
- [[Morphological Typology]] — CJK languages span isolating (Mandarin) to agglutinative (Korean)
- [[SentencePiece]] — best choice for production CJK tokenization
- [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Feature Engineering.md]] — script-specific features

---

## Common Interview Questions

**Q: Why does Chinese-BERT use character-level tokenization instead of word segmentation?**
A: Three reasons: (1) Chinese characters carry strong semantic content individually (unlike Latin letters), so character-level representations are meaningful; (2) word segmentation itself is an error-prone NLP task — errors propagate downstream; (3) BERT's self-attention can implicitly learn word boundaries by attending to adjacent characters, eliminating the need for explicit segmentation. In practice, character-level Chinese-BERT outperforms word-level alternatives on most tasks.

**Q: What is the main challenge of Japanese tokenization that English doesn't have?**
A: Multiple writing systems (Hiragana, Katakana, Kanji, Latin) mixed in single sentences, no spaces between words, and rich morphological agglutination. A single word can span characters from multiple scripts, and the morphological analyzer must simultaneously handle all script types. Proper Japanese tokenization requires a dictionary (MeCab + IPADIC/UniDic) or a trained neural segmenter; simple rules don't work.

---

## Common Mistakes / Gotchas

- **Using whitespace tokenization on Chinese/Japanese**: produces the entire sentence as one token. Always use a dedicated segmenter or character-level tokenization.
- **Setting `character_coverage < 1.0` for Chinese in SentencePiece**: with 0.9995 coverage, some Chinese characters (especially rare ones) become `<unk>`. For Chinese/Japanese/Korean, always set `character_coverage=1.0`.
- **Applying English normalization to CJK**: lowercasing, accent stripping, and stemming rules don't apply to Chinese/Japanese. Applying them corrupts the text.
- **Ignoring Traditional vs Simplified Chinese**: Traditional Chinese (used in Taiwan, Hong Kong) and Simplified Chinese (used in mainland China) have different character sets. Models trained on one may not transfer well to the other without explicit handling.

---

## Further Reading / Paper References

- Cui, Y. et al. (2021). *Pre-Training with Whole Word Masking for Chinese BERT.* [[arxiv:1906.08101]] — Chinese character-level BERT
- Kudo, T. (2018). *SentencePiece.* [[arxiv:1808.06226]] — handles CJK natively
- MeCab: https://taku910.github.io/mecab/ — Japanese tokenizer
- Jieba: https://github.com/fxsjy/jieba — Chinese word segmentation
- Jurafsky & Martin, *Speech and Language Processing* Ch. 2 — tokenisation including CJK
