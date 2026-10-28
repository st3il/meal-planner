# meal-planner

Mittagessen-Wochenplan. Jeden Dienstag um 6:45 Uhr wird automatisch der Plan
für Mittwoch bis Dienstag erstellt und hier gepusht.

## Aufbau

```
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
