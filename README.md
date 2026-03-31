# 🤖 AI Code Reviewer — Hackathon Voorstel

## Het probleem

- Reviews worden **overgeslagen** als het druk is
- **Beveiligingskennis** zit in één persoon, niet in het team
- Incidenten worden **opnieuw geïntroduceerd** door nieuwe developers
- Kleine "quick fixes" krijgen **geen aandacht** — maar zijn net zo gevaarlijk

---

## Het idee

Bij elke **Pull Request** op GitHub wordt automatisch een AI-agent geactiveerd die de code reviewt en commentaar plaatst. Aangedreven door **GitHub Copilot**, aangestuurd door Markdown-bestanden die het team zelf beheert.

**Geen wachten. Geen overgeslagen checks. Consistent op elke PR.**

---

## Hoe werkt het?

```
Developer opent PR
        ↓
GitHub Actions start automatisch
        ↓
Relevante .md bestanden worden ingeladen als context
        ↓
Code-diff + context gaat naar GitHub Copilot
        ↓
Review verschijnt automatisch op de PR
```

---

## Bestandsstructuur

```
.github/
├── workflows/
│   └── ai-review.yml              # De trigger
│
├── reviewer/
│   ├── master-prompt.md           # Hoofdinstructies voor de agent
│   ├── security-rules.md          # Beveiligingsregels
│   ├── architecture.md            # Systeemoverzicht
│   ├── incident-history.md        # Bekende valkuilen
│   │
│   └── languages/
│       ├── python.md
│       ├── javascript.md
│       ├── csharp.md
│       └── yaml.md
│
└── scripts/
    └── review.py
```

> Het script laadt **automatisch de juiste taalbestanden** op basis van wat er in de PR is gewijzigd.

---

## De contextbestanden

Dit is het brein van het systeem. Zonder deze bestanden geeft Copilot een **generieke review**. Mét deze bestanden geeft hij feedback gebaseerd op de kennis van het team.

### `master-prompt.md`
**Vertelt de agent wie hij is en hoe hij moet reviewen.** Stel hier in: ernstniveaus, taal, outputformaat en reviewprioriteiten.

```markdown
Jij bent een senior software engineer die PRs reviewt voor [bedrijfsnaam].
Schrijf in het Nederlands. Prioriteer beveiligingsproblemen altijd.
Gebruik: 🔴 Kritiek | 🟡 Waarschuwing | 🔵 Suggestie
```

---

### `security-rules.md`
**De beveiligingskennis van het team, automatisch toegepast op elke PR.**

- Niet elke developer kent alle risico's — dit bestand wel
- **Geen code nodig** — gewone tekst volstaat
- Nieuwe regel toegevoegd? **Geldt direct voor het hele team**
- Elk reviewcommentaar blijft op de PR staan → **audittrail voor compliance**

Typische inhoud: gevaarlijke functies, auth-regels, AVG-controles, hardcoded secrets, HTTPS-eisen.

---

### `incident-history.md`
**De incidentkennis van de organisatie, omgezet naar live reviewfeedback.**

- Incidentkennis zit nu in mensen — **verdwijnt als ze vertrekken**
- Door het hier te documenteren wordt het **structureel gedeeld** met elke developer
- Een nieuwe developer die hetzelfde patroon introduceert krijgt **direct een waarschuwing**
- Niemand leest postmortems. Dit werkt **automatisch op elke PR**.

Typische inhoud: wat er misging, welk codepatroon het veroorzaakte, wat de agent moet doen.

---

### `architecture.md`
**Geeft de agent context over het systeem** — zodat hij niet in een vacuum beoordeelt.

Typische inhoud: componentoverzicht, ontwerpbeslissingen, grenzen die niet overschreden mogen worden.

---

### `languages/`
**Taalspecifieke regels** — alleen ingeladen als die taal in de PR voorkomt. Gefocust en relevant.

---

## Waarom dit beter is dan bestaande tools

| | Statische analysetool | AI Code Reviewer |
|---|---|---|
| Begrijpt context | ❌ | ✅ |
| Gebruikt teamkennis | ❌ | ✅ |
| Leert van incidenten | ❌ | ✅ |
| Aanpasbaar zonder code | ❌ | ✅ |
| Schaalt mee met het team | ✅ | ✅ |

---

## De workflow

```yaml
name: AI Code Review met Copilot
on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read
  pull-requests: write

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install requests
      - run: python .github/scripts/review.py
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          PR_NUMBER: ${{ github.event.pull_request.number }}
          REPO: ${{ github.repository }}
```

> **Geen extra API-sleutel nodig** — de ingebouwde `GITHUB_TOKEN` volstaat.

---

## Wat bouwen we tijdens de hackathon?

1. **GitHub Actions workflow** — de trigger
2. **Review-script** — diff ophalen, context inladen, Copilot aanroepen
3. **Startset contextbestanden** — master prompt, beveiligingsregels, incidenthistorie
4. **Live demo** — nep-PR met bewuste fouten (SQL-injectie, hardcoded secret, ontbrekende timeout)

---

## Uitbreidingen (als er tijd over is)

- 💬 **Inline commentaar** op specifieke regels via de GitHub Review API
- 🔁 **`/re-review`** — agent opnieuw aanroepen via een PR-comment
- 📝 **Auto-samenvatting** — agent schrijft een beschrijving bij de PR

---

## Samenvatting

> Copilot is de motor. **Het team is de hersenen.**
> De kennis van beveiliging, beheer en architectuur wordt automatisch feedback op elke PR — zonder extra werk, zonder uitzonderingen.
