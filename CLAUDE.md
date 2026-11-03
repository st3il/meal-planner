# Wöchentlicher Essensplan – Anweisungen für Claude

Diese Datei steuert die geplante Aufgabe (Cloud-Routine), die jeden Dienstag um
6:45 Uhr läuft. Der Prompt der Routine klont nur das Repo und verweist hierher;
alle Regeln werden hier gepflegt. Aufbau und Schemata stehen in `README.md`.

Ziel: Mittagessen-Plan für unsere Familie, gültig von morgen (Mittwoch) bis
einschließlich nächsten Dienstag, als JSON ins Repo pushen. Alles auf Deutsch.

## Wochenordner

`<Jahr>/<KW>/` mit KW = ISO-Kalenderwoche des morgigen Mittwochs, ohne führende
Null (z. B. `2026/45/`). Jahr = ISO-Wochenjahr dieses Mittwochs.

## Quellen

- **Rezepte:** `rezepte.json` (Quelle der Wahrheit). Rezepte mit Notiz
  „Name unklar“ ignorieren.
- **Wünsche und Abweichungen:** `<Jahr>/<KW>/wuensche.json`, falls vorhanden
  (Schema in `README.md`). Fehlt die Datei oder sind die Listen leer → keine
  Wünsche bzw. keine Abweichungen.
- **Bisherige Pläne:** `plan.json` der Vorwochen (zur Abwechslung abgleichen).
- **Kalender:** Google Kalender (primärer Kalender) über den
  Google-Calendar-Connector, Zeitraum Mi–Di.

## Wer isst mittags zu Hause

Feste Wochenroutine (steht nicht im Kalender):

| Tag | Wer |
| --- | --- |
| Mi | nur Sascha (frei, geht einkaufen) |
| Do | nur Sascha (Homeoffice; Anke im Büro) |
| Fr | Sascha und Anke |
| Sa/So | Sascha und Anke |
| Mo | nur Anke (Sascha im Büro) |
| Di | nur Anke (Homeoffice; Sascha im Büro) |

Reihenfolge, was gilt:

1. **`zuHause` in `wuensche.json`** ersetzt die Routine für den jeweiligen Tag
   komplett und hat Vorrang vor Kalender und Wünschen.
2. **Kalender** für alle übrigen Tage auf Ausnahmen prüfen: Urlaub, freie Tage,
   Feiertage, Termine über Mittag (ca. 11:30–14:00), Reisen, Besuch, Essen
   auswärts. Termine der Kinder am Nachmittag (Tanzen, Turnen usw.) sind keine
   Ausnahme. Bin ich an einem Bürotag ausnahmsweise zu Hause → `wer` anpassen.
3. Sonst die feste Routine.

Ist niemand da (`"wer": []`): Tag mit `"rezeptId": null`, `"gericht": null`,
`"wer": []` und Grund im `hinweis`; keine Reste dorthin, nichts auf die
Einkaufsliste. Jede Abweichung von der Routine im `hinweis` nennen.

## Wünsche

Wünsche haben Vorrang vor den Auswahlregeln unten (nicht vor `zuHause`).

- Jeder Eintrag gilt für den genannten `tag` bzw. `datum`; sind beide `null`,
  an einem passenden Tag einplanen.
- `rezeptId` gesetzt → genau dieses Rezept. Sonst `wunsch` auswerten: Richtung
  (z. B. „was mit Hackfleisch“, „Suppe“) → am besten passendes Rezept wählen;
  neues Gericht → trotzdem einplanen und als neues Rezept in `rezepte.json`
  ergänzen (eindeutige id im Slug-Stil: Kleinbuchstaben, ä→ae, ö→oe, ü→ue,
  ß→ss, Bindestriche; Zutaten selbst ableiten).
- Ein Wunsch darf auch die Fleisch-Regel brechen.
- Erfüllte Wünsche im `hinweis` mit „Wunsch: <Text oder Rezeptname>“
  kennzeichnen. Kann ein Wunsch nicht erfüllt werden (z. B. niemand zu Hause),
  in der Nachricht sagen warum.
- `wuensche.json` nicht verändern oder löschen; sie bleibt als Verlauf.

## Auswahlregeln

- **Ernährung:** „vegetarisch“ und „fisch“ an jedem Tag erlaubt. „fleisch“ nur
  am Mittwoch – möglich, kein Muss. Reste am Donnerstag nie von einem
  Fleischgericht – wird mittwochs Fleisch gekocht, für Donnerstag ein eigenes
  Gericht ohne Fleisch planen. Fisch höchstens 1–2 Mal pro Woche.
- Keine Wiederholung innerhalb der Woche; Rezepte bevorzugen, deren
  `zuletztGegessen` `null` ist oder mehr als 3 Wochen zurückliegt.
- Abwechslung: nicht zwei Nudel-, Reis- oder Kartoffelgerichte direkt
  hintereinander (Tage ohne Essen zählen nicht als Unterbrechung).
- „aufwendig“ nur am Wochenende oder an freien Tagen; werktags „schnell“ oder
  „mittel“.
- Tage mit nur einer Person: bevorzugt Reste vom Vortag (am Vortag „Doppelte
  Menge – Rest für <Tag>“ in `hinweis`), sonst ein Rezept mit
  `"fuerEinePerson": true`. Typisch: Mi kochen → Do Reste; So doppelt → Mo
  Reste; Di schnelles Gericht für eine Person. Reste nur von Gerichten, die
  sich gut aufwärmen lassen.

## Ausgabe im Repo

1. `<Jahr>/<KW>/plan.json` schreiben (überschreiben, falls vorhanden), exakt im
   Schema der bisherigen Pläne:
   ```json
   {
     "kw": 45, "jahr": 2026, "von": "YYYY-MM-DD", "bis": "YYYY-MM-DD", "erstellt": "YYYY-MM-DD",
     "tage": [
       {"datum": "YYYY-MM-DD", "tag": "Mi", "wer": ["Sascha"], "rezeptId": "<id aus rezepte.json>", "gericht": "<Name, bei Resten 'Reste <Name>'>", "reste": false, "hinweis": "<Text oder null>"}
     ],
     "einkaufsliste": [
       {"artikel": "Hackfleisch", "menge": "250 g", "bereich": "Fleisch & Fisch"}
     ]
   }
   ```
   Bereiche: „Obst & Gemüse“, „Kühlregal“, „Fleisch & Fisch“, „Trockenwaren &
   Konserven“, „TK“, „Sonstiges“. Mengen grob passend zur Personenzahl und zu
   den Resten; `menge` darf `null` sein. Grundvorrat weglassen: Salz, Pfeffer,
   Öl, Zucker, gängige Gewürze, Brühe.
2. In `rezepte.json` bei jedem eingeplanten Rezept `zuletztGegessen` auf das
   geplante Datum setzen (bei Resten zählt der Kochtag) und `aktualisiert` auf
   heute. Sonst nichts ändern (außer neu aufgenommenen Rezepten aus Wünschen).
   Formatierung beibehalten: `json.dump(..., ensure_ascii=False, indent=2)`
   plus Zeilenumbruch am Ende.
3. Für die Folgewoche eine leere Vorlage `<Jahr>/<KW+1>/wuensche.json` anlegen
   (`{"kw": .., "jahr": .., "wuensche": [], "zuHause": []}`), falls sie noch
   nicht existiert – Jahreswechsel per ISO-Woche beachten.
4. Alle geänderten Dateien mit Python auf gültiges JSON prüfen und alle
   `rezeptId` gegen `rezepte.json` abgleichen.
5. Ein Commit auf `main` mit der Nachricht „Wochenplan KW <KW>
   (<TT.MM.>–<TT.MM.>)“, dann `git push origin main`. Bei abgelehntem Push:
   `git pull --rebase origin main` und einmal erneut pushen.
6. Nichts in Google Drive oder Google Sheets schreiben.

## Nachricht an mich (am Ende)

- Die Gerichte pro Tag, eine Zeile je Tag
- Welche Wünsche berücksichtigt wurden (oder dass keine da waren), Abweichungen
  aus `zuHause` und erkannte Kalender-Ausnahmen
- „Für Bring:“ und darunter alle Einkaufsartikel kommagetrennt ohne Mengen in
  einer Zeile
- Den Link `https://github.com/st3il/meal-planner/blob/main/<Jahr>/<KW>/plan.json`
- Den Hinweis, wohin Wünsche und Abweichungen für nächste Woche gehören:
  `<Jahr>/<KW+1>/wuensche.json`

## Fehlerfälle

- Kalender nicht erreichbar (bei Kontingent-Fehlern einmal erneut versuchen):
  trotzdem nach Routine und `zuHause` planen und in der Nachricht sagen, dass
  er gefehlt hat.
- Push scheitert: den Plan vollständig in der Nachricht ausgeben und den
  Fehler nennen.
