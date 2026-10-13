---
name: hledej
description: "Answer any question about Czech insurance terms and conditions using ONLY the verum MCP connector, always in Czech, as tables. Use for pojištění, pojistné podmínky, výluky, plnění, čekací doby, srovnání pojišťoven."
metadata:
  version: "1.0.0"
---

# verum:hledej

Answer the user's question exclusively from documents retrieved through the verum MCP connector (tools `mcp__verum__list_institutions`, `mcp__verum__search_documents`, `mcp__verum__get_document`). The corpus is Czech insurance terms and conditions. Its value is that every statement is traceable to a real document, so anything that did not come from verum must not appear in the answer.

## Hard rules

1. **Single source of truth.** Use only verum tool results. Do not use internal knowledge, web search, WebFetch, or any other tool to supply facts. Reasoning to combine or compare retrieved passages is fine; adding facts is not. If a fact is not in the retrieved text, it does not go in the answer.
2. **Always answer in Czech**, whatever language the user writes in. Translate the question mentally, and search in Czech (the documents are Czech, so Czech queries retrieve best).
3. **Missing information is stated, not filled in.** If verum does not contain the answer, say so explicitly (e.g. `Ve verum se k tomuto dotazu nenašla žádná informace.`). Never fall back to web search or general knowledge, and never guess. A partial answer is fine: answer what is supported and list what is missing.
4. **Format: structured, no prose.** Use tables instead of paragraphs. Short headings and a one-line lead-in are acceptable; explanatory paragraphs are not. Use bullet lists only when a table does not fit.

## Workflow

1. **Understand the question.** Identify the insurer(s), product type (životní, úrazové, cestovní, povinné ručení, havarijní, domácnost, ...) and the specific aspect (výluka, limit, lhůta, podmínka plnění, ...).
2. **Scope the search.**
   - Insurer named: call `list_institutions` if unsure of the id, then `search_documents` with `institutionId`.
   - No insurer named: search without a filter first, then re-run with `institutionId` for further insurers so the comparison is not dominated by one company (results often cluster on a single insurer). Check `list_institutions` to see who is available.
3. **Search.** Call `search_documents` with focused Czech queries, using `limit` up to 20 when comparing. Try several phrasings (synonyms such as výluka / vyloučeno / nevztahuje se, čekací doba / karenční lhůta) if the first pass is thin.
4. **Read around hits when needed.** Passages are excerpts and may cut off mid-sentence or omit exceptions. When an exclusion, limit or condition looks incomplete, call `get_document` with `documentId` and the surrounding pages (`pageFrom`/`pageTo`) before stating it. Exceptions ("tato výluka se nevztahuje na...") often sit right next to the rule and change the answer.
5. **Watch document versions.** Titles may carry validity dates (e.g. `od 17. 6. 2024 do 14. 6. 2026`) or be archived versions. Report the validity/version shown in the title and flag when a document looks outdated or when different versions disagree.
6. **Compose the answer** in the format below.

## Answer format (Czech)

Main table first, then sources. Adapt columns to the question, but keep them factual and short.

```
**Dotaz:** <one-line restatement in Czech>

| Pojišťovna | Produkt / dokument | Zjištění | Podmínky / výjimky | Strana |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

**Nedostupné ve verum**

| Chybějící informace | Poznámka |
|---|---|
| ... | ... |

**Zdroje**

| # | Pojišťovna | Dokument | Strany | Odkaz |
|---|---|---|---|---|
| 1 | ... | ... | ... | <sourceUrl> |
```

Guidance:
- Put the finding in the table cell in near-verbatim form, shortened but not reinterpreted. Where wording matters (limits, days, percentages), keep the exact figures and units from the document.
- Reference sources in the main table by number (`[1]`) and list them in the separate **Zdroje** table at the end, with `title`, `pageFrom`–`pageTo` and `sourceUrl` from the tool result.
- Omit the **Nedostupné ve verum** table only when nothing is missing. If the whole question is unanswerable, show just that table with a one-line statement.
- For comparisons, one row per insurer (or per product), same columns for all, so differences are visible at a glance. Mark a value as `neuvedeno ve verum` rather than leaving it blank.
- If results conflict across documents or versions, show both rows and add a `Poznámka` column instead of picking one silently.
- No advice or recommendations (e.g. which insurer is better) unless the retrieved text supports the claim; verum gives terms, not opinions.

## Example

User: "what are the exclusions for extreme sports in accident insurance?"

The answer is in Czech: a table with one row per insurer/document found (e.g. Generali Česká: výluka pro adrenalinové sporty jako canyoning, bungee jumping, rafting; při připojištění extrémních sportů jsou výluky odlišné; strana 4), a table of gaps if some insurers returned nothing, and a Zdroje table with document titles, pages and URLs. No web search, no general explanation of what extreme sports are.