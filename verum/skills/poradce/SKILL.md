---
name: poradce
description: "Pojišťovací poradce nad databází verum: krátce vyzpovídá uživatele (typ pojištění + 3 kola doplňujících otázek o pojišťovaném objektu – osobě, majetku, autě, firmě) a pak ve verum najde 3 pojišťovny a produkty, jejichž pojistné podmínky objektu nejlépe odpovídají. Použij vždy, když uživatel chce doporučit, vybrat nebo najít vhodné pojištění pro svůj konkrétní případ (byt, dům, chata, auto, život, firma, povinné ručení, havarijní), ptá se „jaké pojištění si vzít“, „kterou pojišťovnu zvolit“, „co je pro mě nejlepší“, nebo napíše /verum:poradce – i když nezmíní verum. Pro obecné dotazy na konkrétní podmínky či výluky bez výběru produktu stačí skill verum:hledej."
metadata:
  version: "1.0.0"
---

# verum:poradce

Rozšíření skillu **verum:hledej**. Cílem je převést nejasné „jaké pojištění si mám vzít?“ na profil konkrétního pojišťovaného objektu a ten pak porovnat s pojistnými podmínkami ve verum. Doporučení stojí jen na tom, co dokumenty skutečně říkají – proto je pohovor důležitý: čím přesnější profil, tím ostřejší rozdíly mezi produkty.

## Krok 0 – Načti pravidla verum

Před čímkoli dalším načti skill `verum:hledej` (nástrojem Skill; podle prostředí může být uveden i jen jako `hledej`). Pokud to nejde, přečti jeho `SKILL.md` přímo – leží ve stejném pluginu v `../hledej/SKILL.md` vůči této složce. Všechna jeho pravidla platí i zde a mají přednost: jediný zdroj faktů je verum, odpovědi vždy česky, tabulky místo odstavců, chybějící informace se přiznávají, zdroje se uvádějí. Tento skill k nim přidává jen pohovor a hodnocení vhodnosti.

Jediná vědomá výjimka z verum pravidla „bez doporučení“: tady doporučujeme, ale **výhradně jako shodu podmínek s profilem**, kde každé kritérium je doložené citací z dokumentu. Nikdy ne „tahle pojišťovna je lepší“ obecně, nikdy cena (verum ceny neobsahuje), nikdy hodnocení služeb či pověsti.

## Krok 1 – Pohovor (1 + 3 kola)

Pokud uživatel už v úvodní zprávě něco prozradil („chci pojistit byt v Praze“), nepokládej otázky, na které už odpověděl – předvyplň je a ptej se jen na zbytek.

Kde je k dispozici nástroj **AskUserQuestion**, použij ho (max. 4 otázky na kolo, 2–4 možnosti; uživatel může vždy napsat vlastní odpověď). Jinak polož otázky stručně v textu jako číslovaný seznam s nabídnutými možnostmi. Vždy česky.

**Kolo 1 – typ pojištění** (jedna otázka):
- Pojištění majetku (byt, dům, chata, garáž, domácnost)
- Životní pojištění
- Povinné ručení
- Havarijní pojištění
- Podnikatelské pojištění (firma, živnost)

Pokud uživatel napíše něco jiného (cestovní, úrazové…), přizpůsob se a kola postav analogicky.

**Kola 2–4 – doplňující otázky.** Otázky pro každý typ jsou v `references/dotazniky.md` – přečti jen sekci pro zvolený typ. Struktura je vždy stejná:
- **Kolo 2 – co přesně je objekt** (druh nemovitosti / kdo je pojištěný / jaké vozidlo / jaký obor),
- **Kolo 3 – vlastnosti a rizika objektu** (lokalita, velikost, stáří, záplavová zóna, zabezpečení, zdravotní stav, nájezd…),
- **Kolo 4 – priority a požadavky** (co je pro uživatele nutnost vs. bonus, výše limitů, spoluúčast, asistence).

Otázky v kolech 3 a 4 přizpůsob odpovědím z předchozích kol (garáž nepotřebuje otázku na podlaží bytu; pojištění pro 70letého se liší od pojištění pro rodiče malých dětí). Ptej se jen na to, co reálně změní, který produkt vyhraje – každá otázka musí mít odpovídající kritérium, které se dá ve verum ověřit.

Po kole 4 shrň profil v tabulce **Profil objektu** (atribut | hodnota | proč je důležitý) a rozděl kritéria na **nutné** (pokud produkt nesplní, vypadává) a **žádoucí** (body navíc). Pokud uživatel neodpověděl na něco zásadního, označ to jako předpoklad a pokračuj – nezdržuj dalším kolem.

## Krok 2 – Hledání ve verum

1. Zavolej `list_institutions` a urči, které pojišťovny jsou pro daný typ relevantní (např. NN a MetLife jsou hlavně životní; Direct, ČPP, Kooperativa, Allianz, Generali, UNIQA, ČSOB, Slavia, HVP mají majetek i auta).
2. Najdi **kandidátní produkty**: dotaz na typ produktu bez filtru, pak cíleně po jednotlivých pojišťovnách s `institutionId`, aby výsledky nedominovala jedna firma. Cílem je pokrýt aspoň 5–6 pojišťoven, pokud je verum má.
3. **Ověř aktuálnost.** Z názvů dokumentů odvoď platnost. Zaniklé/archivní verze (např. „zaniklá“, „archiv“, „platné od … do …“ s koncem v minulosti) mezi kandidáty nepouštěj; pokud jiná verze není, zařaď je s jasným varováním.
4. **Pro každé kritérium z profilu** spusť cílené dotazy (šablony dotazů jsou u každého typu v `references/dotazniky.md`). Opakuj s `institutionId` pro každého kandidáta, aby nevypadl jen proto, že nebyl v top výsledcích.
5. U rozhodujících míst (výluky, limity, čekací doby, záplavové zóny Q10/Q20/Q50, spoluúčasti) dočti okolí přes `get_document` – výjimky z výluk často mění výsledek.

Rozpočet: pracuj, dokud nemáš u top kandidátů doložené všechny nutné kritérium – typicky 15–30 volání. Raději méně kandidátů do hloubky než mnoho povrchně.

## Krok 3 – Hodnocení

Pro každého kandidáta (pojišťovna × produkt) vyplň matici kritérií:
- ✅ **splněno** – dokument to výslovně kryje / umožňuje,
- 🟡 **částečně** – kryto s omezením (nízký limit, delší čekací doba, jen jako připojištění),
- ❌ **nesplněno** – výslovná výluka nebo nemožnost,
- ❔ **neuvedeno ve verum** – nenašlo se; nepočítá se jako ❌, ale snižuje jistotu.

Pořadí: nejdřív vyřaď produkty s ❌ u nutného kritéria. Zbytek seřaď podle počtu ✅/🟡 u nutných, pak u žádoucích; při shodě vyhrává ten s méně ❔ (lépe doložený). Pokud méně než 3 produkty projdou, ukaž kolik jich prošlo a proč ostatní vypadly – nedoplňuj pořadí slabými kandidáty bez upozornění.

## Krok 4 – Výstup (česky, tabulky)

```
**Profil objektu**
| Atribut | Hodnota | Typ kritéria |

**Top 3 doporučení**
| # | Pojišťovna | Produkt / dokument (platnost) | Proč sedí na váš objekt | Hlavní omezení | Zdroj |

**Porovnání kritérií**
| Kritérium | Produkt 1 | Produkt 2 | Produkt 3 |   ← buňky: ✅/🟡/❌/❔ + krátká citace a číslo, odkaz [n]

**Vyřazené produkty**  (jen pokud nějaké)
| Pojišťovna | Produkt | Důvod vyřazení | Zdroj |

**Nedostupné ve verum**
| Chybějící informace | Poznámka |

**Zdroje**
| # | Pojišťovna | Dokument | Strany | Odkaz |
```

Pod tabulkami jedna řádka: *Pořadí vychází pouze z pojistných podmínek ve verum; nezohledňuje cenu ani individuální nabídku. Před sjednáním si ověřte aktuální podmínky a cenu u pojišťovny.*

Pravidla pro buňky: čísla, limity, dny a pásma přepisuj přesně z dokumentu; ve sloupci „Proč sedí“ vždy odkazuj na konkrétní atribut profilu („kryje záplavu i v zóně Q20 – váš byt je v Q20“). Nic, co nepochází z verum, se v tabulkách neobjeví.
