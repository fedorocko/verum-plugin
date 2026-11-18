# verum – plugin pro Claude

Plugin zpřístupňuje v Claude pojistné podmínky českých pojišťoven z databáze verum. Obsahuje dva skilly a konektor (MCP server) k verum.

| Příkaz | Co dělá |
|---|---|
| `/verum:hledej` | Vyhledá v pojistných podmínkách konkrétní informace (výluky, limity, čekací doby, srovnání pojišťoven) a odpoví česky v tabulkách se zdroji. |
| `/verum:poradce` | Zeptá se na typ pojištění a pojišťovaný objekt a pak najde 3 pojišťovny a produkty, jejichž podmínky mu nejlépe odpovídají. |

## Co konektor poskytuje

Přes konektor Claude vyhledává a čte pojistné podmínky a další dokumenty českých pojišťoven v databázi verum. Všechny nástroje data jen čtou, nic nemění.

<!-- Jeden řádek na každý nástroj MCP serveru. Když server nástroj přidá, přejmenuje nebo odebere, upravte tuto tabulku i English summary. -->

| Nástroj | Co poskytuje |
|---|---|
| `list_institutions` | Všechny instituce, jejichž dokumenty lze prohledávat: ID, název, slug a kategorie. Podle ID filtrují `search_documents` a `list_new_documents`. |
| `search_documents` | Pasáže, které významově nejlépe odpovídají dotazu, s dokumentem a stranou, odkud pocházejí. Lze omezit na jednu instituci. Výchozí počet pasáží je 10, nejvýše 20. Nejlépe funguje s dotazem v češtině. |
| `get_document` | Text jednoho dokumentu po stranách, s názvem, odkazem na zdroj a celkovým počtem stran. Výchozí rozsah je 10 stran. Slouží k dočtení okolí nalezené pasáže. |
| `list_new_documents` | Dokumenty, které se objevily na webech pojišťoven za posledních N dní (výchozí 14, nejvýše 365), od nejnovějších, po 50. „Objevily se“ znamená, že je naše pondělní procházení webů našlo poprvé; nejde o datum vydání ani účinnosti. Zahrnuje i formuláře a zprávy. |

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

A Claude plugin for Czech insurance terms and conditions from the verum database. It bundles two skills and the verum MCP connector:

- `/verum:hledej` looks up specific terms (exclusions, limits, waiting periods, insurer comparisons) and answers in Czech, as tables with sources.
- `/verum:poradce` interviews you about what you want to insure, then finds the 3 insurers and products whose terms fit best.

The connector gives Claude four read-only tools: `list_institutions` (the searchable institutions and their IDs), `search_documents` (the passages that best match a question by meaning), `get_document` (the text of one document, by page) and `list_new_documents` (the documents first found on insurers' websites in the last N days).

Install on claude.ai or the desktop app via **Customize → Plugins → Add → Add marketplace** with `fedorocko/verum-plugin`, then open the plugin's **Connectors** tab and click **Connect**. In Claude Code, run `/plugin marketplace add fedorocko/verum-plugin` and `/plugin install verum@verum-marketplace`. The skills load without the connector but cannot answer until it shows **Connected**.
