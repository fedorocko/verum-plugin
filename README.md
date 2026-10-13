# verum – plugin pro Claude

Plugin zpřístupňuje v Claude pojistné podmínky českých pojišťoven z databáze verum. Obsahuje dva skilly a konektor (MCP server) k verum.

| Příkaz | Co dělá |
|---|---|
| `/verum:hledej` | Vyhledá v pojistných podmínkách konkrétní informace (výluky, limity, čekací doby, srovnání pojišťoven) a odpoví česky v tabulkách se zdroji. |
| `/verum:poradce` | Zeptá se na typ pojištění a pojišťovaný objekt a pak najde 3 pojišťovny a produkty, jejichž podmínky mu nejlépe odpovídají. |

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

Install on claude.ai or the desktop app via **Customize → Plugins → Add → Add marketplace** with `fedorocko/verum-plugin`, then open the plugin's **Connectors** tab and click **Connect**. In Claude Code, run `/plugin marketplace add fedorocko/verum-plugin` and `/plugin install verum@verum-marketplace`. The skills load without the connector but cannot answer until it shows **Connected**.
