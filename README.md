# meal-planner

Mittagessen-Wochenplan. Jeden Dienstag um 6:45 Uhr wird automatisch der Plan
für Mittwoch bis Dienstag erstellt und hier gepusht. Die Regeln dafür stehen in
`CLAUDE.md` – dort anpassen, nicht im Prompt der geplanten Aufgabe.

## Aufbau

```
CLAUDE.md                 Anweisungen für die geplante Aufgabe
rezepte.json              alle Rezepte (Quelle der Wahrheit)
<Jahr>/<KW>/plan.json     Wochenplan, KW = ISO-Kalenderwoche des Mittwochs
<Jahr>/<KW>/wuensche.json Wünsche für diese Woche (optional)
```

## Wünsche eintragen

Datei `<Jahr>/<KW>/wuensche.json` anlegen oder ergänzen, **bevor** der Plan
am Dienstag davor erstellt wird. Beispiel für KW 45 (04.11.–10.11.2026):

```json
{
  "kw": 45,
  "jahr": 2026,
  "wuensche": [
    { "tag": "Mi", "datum": null, "rezeptId": null, "wunsch": "was mit Hackfleisch", "wer": "Sascha" },
    { "tag": null, "datum": "2026-11-07", "rezeptId": "kaesespaetzle", "wunsch": null, "wer": "Anke" },
    { "tag": null, "datum": null, "rezeptId": null, "wunsch": "mal wieder Suppe", "wer": null }
  ],
  "zuHause": [
    { "tag": "Fr", "datum": null, "wer": ["Anke"], "grund": "Sascha im Büro" },
    { "tag": "Sa", "datum": null, "wer": [], "grund": "unterwegs" }
  ]
}
```

| Feld | Bedeutung |
| --- | --- |
| `tag` | `Mi`, `Do`, `Fr`, `Sa`, `So`, `Mo`, `Di` – oder `null` |
| `datum` | `YYYY-MM-DD` – alternativ zu `tag`; beide `null` = irgendein passender Tag |
| `rezeptId` | konkretes Rezept aus `rezepte.json` – oder `null` |
| `wunsch` | Freitext (Richtung oder neues Gericht) – oder `null` |
| `wer` | optional, wer sich das wünscht |

Mindestens `rezeptId` oder `wunsch` muss gesetzt sein. Wünsche haben Vorrang
vor allen anderen Regeln (auch vor „Fleisch nur mittwochs“). Ein neues Gericht
wird automatisch in `rezepte.json` aufgenommen.

## Abweichungen: wer ist zu Hause

Normalerweise gilt die feste Wochenroutine (Mi/Do Sascha, Fr–So beide,
Mo/Di Anke). In `zuHause` stehen Tage, an denen das anders ist. Jeder Eintrag
hat `tag` oder `datum` (wie oben), `wer` (wer an dem Tag mittags isst) und
optional einen `grund`. `wer` ersetzt die Routine für diesen Tag komplett.

- `"wer": []` = kein Essen (z. B. unterwegs): kein Gericht, keine Reste dorthin,
  nichts auf die Einkaufsliste. Im Plan steht der Tag mit `"rezeptId": null`,
  `"gericht": null`, `"wer": []` und dem Grund als `hinweis`.
- `zuHause` hat Vorrang vor dem Kalender und vor Wünschen für denselben Tag.
