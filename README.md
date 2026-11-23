# verum – plugin pro Claude

Plugin zpřístupňuje v Claude pojistné podmínky českých pojišťoven z databáze verum. Obsahuje tři skilly a konektor (MCP server) k verum.

| Příkaz | Co dělá |
|---|---|
| `/verum:hledej` | Vyhledá v pojistných podmínkách konkrétní informace (výluky, limity, čekací doby, srovnání pojišťoven) a odpoví česky v tabulkách se zdroji. |
| `/verum:poradce` | Zjistí, jaké pojištění potřebujete, a porovná podmínky relevantních pojišťoven s vašimi kritérii. Produkty neřadí a nevybírá vítěze. |
| `/verum:novinky` | Najde nově objevené dokumenty, porovná je se staršími dokumenty stejné pojišťovny a produktu a odpoví česky přehledem top 5 a seznamem podle pojišťoven a produktů se zdroji. |

## Co konektor poskytuje

Přes konektor Claude vyhledává a čte pojistné podmínky a další dokumenty českých pojišťoven v databázi verum. Všechny nástroje data jen čtou, nic nemění.

<!-- Jeden řádek na každý nástroj MCP serveru. Když server nástroj přidá, přejmenuje nebo odebere, upravte tuto tabulku i English summary. -->

| Nástroj | Co poskytuje |
|---|---|
| `list_institutions` | Všechny instituce, jejichž dokumenty lze prohledávat: ID, název, slug a kategorie. Podle ID filtrují `search_documents`, `list_new_documents` a `list_document_classes`. |
| `list_document_types` | Všechny platné kódy typů dokumentů s českým názvem, také typy, které teď nemají žádný dokument. Podle kódů filtrují `search_documents` a `list_document_classes`. |
| `list_product_types` | Všechny platné kódy typů produktů s českým názvem a údajem, zda jde o produkt pro firmy, nebo pro občany. Podle kódů filtrují `search_documents` a `list_document_classes`. |
| `list_document_classes` | Počty prohledávatelných dokumentů podle typu dokumentu a typu produktu, také po jednotlivých institucích. Lze omezit na instituci, typy dokumentů a typy produktů. |
| `search_documents` | Pasáže, které významově nejlépe odpovídají dotazu, s dokumentem, stranou a dnem, kdy jsme dokument poprvé našli. Lze omezit na jednu instituci, na typy dokumentů a produktů (filtr vynechá dokumenty bez klasifikace) a na dokumenty poprvé nalezené před zadaným dnem (`until`). Výchozí počet pasáží je 10, nejvýše 20. Nejlépe funguje s dotazem v češtině. |
| `get_document` | Text jednoho dokumentu po stranách, s názvem, odkazem na zdroj a celkovým počtem stran. Výchozí rozsah je 10 stran. Slouží k dočtení okolí nalezené pasáže. |
| `list_new_documents` | Dokumenty, které se objevily na webech pojišťoven za posledních N dní (výchozí 14, nejvýše 365), od nejnovějších, po 50. „Objevily se“ znamená, že je naše pondělní procházení webů našlo poprvé; nejde o datum vydání ani účinnosti. Zahrnuje i formuláře a zprávy. U každého dokumentu uvádí typ dokumentu a typy produktů. |

Nástroje nevoláte sami. Zeptejte se Claude, například „Jaké výluky má úrazové pojištění pro extrémní sporty?“, a Claude zavolá nástroje, které potřebuje.

## Instalace

### claude.ai a desktopová aplikace Claude

1. Otevřete **Customize → Plugins → Add → Add marketplace**.
2. Zadejte `fedorocko/verum-plugin` a přidejte plugin **verum**.
3. V pluginu otevřete záložku **Connectors** a klikněte na **Connect**. Přihlásíte se k verum (OAuth).

### Claude Code

```
/plugin marketplace add fedorocko/verum-plugin
/plugin install verum@verum-marketplace
```

K verum se pak přihlásíte příkazem `/mcp`.

## Připojení k verum

Skilly se načtou i bez konektoru, ale odpovídat mohou až ve chvíli, kdy je u konektoru verum stav **Connected**.

## English summary

A Claude plugin for Czech insurance terms and conditions from the verum database. It bundles three skills and the verum MCP connector:

- `/verum:hledej` looks up specific terms (exclusions, limits, waiting periods, insurer comparisons) and answers in Czech, as tables with sources.
- `/verum:poradce` finds out which insurance you need, then compares the terms of the relevant insurers against your criteria. It does not rank products or name a winner.
- `/verum:novinky` reports what is new: it compares newly found documents with the older documents of the same insurer and product.

The connector gives Claude seven read-only tools: `list_institutions` (the searchable institutions and their IDs), `list_document_types` and `list_product_types` (every valid class code with its label), `list_document_classes` (document counts by class and institution), `search_documents` (the passages that best match a question by meaning, with optional class and date filters), `get_document` (the text of one document, by page) and `list_new_documents` (the documents first found on insurers' websites in the last N days, with their classes).

Install on claude.ai or the desktop app via **Customize → Plugins → Add → Add marketplace** with `fedorocko/verum-plugin`, then open the plugin's **Connectors** tab and click **Connect**. In Claude Code, run `/plugin marketplace add fedorocko/verum-plugin` and `/plugin install verum@verum-marketplace`. The skills load without the connector but cannot answer until it shows **Connected**.
