---
name: novinky
description: "Report what is new in Czech insurance documents on pojistnainteligence.cz. It reads the newly found documents, compares them with the older documents of the same insurer and product, and answers in Czech with a top 5 table and a list per insurer and product, with sources. Use for: novinky, co je nového, změny v pojistných podmínkách, nové dokumenty pojišťoven."
metadata:
  version: "1.0.0"
---

# verum:novinky

Find what the new documents add or change compared with the older documents.

## Hard rules

<!-- Rules 1, 2, 3 and 5 are the same text in hledej, poradce and novinky. Change all 3. -->

1. **Single source of truth.** Use only verum tool results. Do not use internal knowledge, web search, WebFetch, or any other tool to supply facts. Reasoning to combine or compare retrieved passages is fine; adding facts is not. If a fact is not in the retrieved text, it does not go in the answer.
2. **Always answer in Czech**, whatever language the user writes in. Translate the question mentally, and search in Czech (the documents are Czech, so Czech queries retrieve best).
3. **Missing information is stated, not filled in.** If verum does not contain the answer, say so explicitly (e.g. `Na pojistnainteligence.cz se k tomuto dotazu nenašla žádná informace.`). Never fall back to web search or general knowledge, and never guess. A partial answer is fine: answer what is supported and list what is missing.
4. **Format.** Use the answer format below. Each news item has 1–2 sentences.
5. **The connector's answer rules win.** The verum connector sends its answer rules in its server instructions and at the end of the description of each tool that returns document text. Follow them in every answer. Where this skill says something else, the answer rules win.

## Workflow

1. **List the new documents.** Call `list_new_documents`, with `days` or `institutionId` if the user gave them. Read every page with `offset` until `truncated` is false. Keep `since`.
2. **Group them** by institution, product type and document type. Call `list_product_types` and `list_document_types`. Show their labels in the answer, never the codes.
3. **Pick at most 15 documents to analyse.** Read the document type labels. Prefer the types that can change what a client is covered for or must do. Do not analyse a document with no class. Count every document that you skip.
4. **Read each new document** with `get_document`. Note its main coverage, exclusions, limits and waiting periods.
5. **Find the older version.** Call `search_documents` with `institutionId`, `until: <since>`, and the document's `productTypes` and `documentTypes`. Take the closest match with the newest `firstSeenAt`.
6. **Compare the same topics** in the old and new text. Use `search_documents` and `get_document`.
7. **Give each document one verdict:**
   - `Nové`: the search in step 5 finds no older document on this topic.
   - `Změna`: the older text says something different. Cite both.
   - `Beze změny`: the content is the same. This is not news.
   - `Neověřeno`: you could not compare the two texts.
8. **Pick the top 5:** first a change to coverage, exclusions, limits or waiting periods, then a new product, then the rest. Show fewer if there are fewer.

## Answer format (Czech)

```
**Novinky od <since>:** <n> nových dokumentů, analyzováno <m>.

| # | Pojišťovna | Typ produktu | Co je nové nebo změněné | Zdroj |
|---|---|---|---|---|
| 1 | ... | ... | **Změna:** <1–2 sentences> | [1][2] |

# <Pojišťovna>

## <Typ produktu>

- **Změna:** <1–2 sentences> [1][2]
- **Nové:** <1–2 sentences> [3]

**Neověřeno a neanalyzováno**

| Pojišťovna | Dokument | Důvod |
|---|---|---|

**Zdroje**

| # | Pojišťovna | Dokument | Článek | Strany | Odkaz |
|---|---|---|---|---|---|

Novinky zahrnují dokumenty, které jsme našli na nové adrese. Dokument nahrazený na stejné adrese zde není vidět.
```

List each document once, under its most specific product. `Beze změny` documents appear only in the count.
