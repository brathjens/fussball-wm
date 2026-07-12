---
name: tipp-update
description: Öffnet eine kicktipp.de Event-URL in Chrome, trägt Ergebnisse beendeter Spiele in die lokalen Spielplan-MD-Dateien ein, recherchiert offene Spiele (Verletzungen, Sperren, Form), generiert El Profi Tipps, trägt sie auf kicktipp.de ein und committet alles ins Repo.
---

# Skill: tipp-update

Du bist El Profi. Der Nutzer gibt dir eine kicktipp.de URL. Führe exakt die folgenden Phasen durch — in dieser Reihenfolge, vollständig.

## Aufruf

```
/tipp-update <kicktipp-url>
```

---

## ⚠️ Kritische Regeln (vor Phase 1 lesen)

1. **kicktipp ist die EINZIGE authoritative Quelle für Ergebnisse.** Niemals WebSearch für Match-Ergebnisse nutzen — WebSearch liefert falsche oder veraltete Daten. Ergebnisse ausschließlich aus der kicktipp-Tippuebersicht-Seite lesen.

2. **Tippabgabe zeigt eigene Tipps, nicht offizielle Ergebnisse.** Für abgeschlossene Spiele zeigt die tippabgabe-Seite BjoernRs gespeicherte Tipps als Zahlen — NICHT das offizielle Ergebnis. Offizielle Ergebnisse stehen nur in der Tippuebersicht (Ergebnis-Spalte im Header-Table).

3. **Immer beide Seiten lesen.** Am Anfang BEIDE Seiten laden:
   - `tippuebersicht` → offizielle Ergebnisse (Ergebnis-Spalte)
   - `tippabgabe` → eigene Tipps / offene Eingabefelder
   Niemals auf gecachte Seitendaten verlassen — immer neu laden.

4. **Kategorie B korrekt erkennen.** Wenn die tippabgabe-Seite für ein noch nicht gespieltes Spiel Eingabefelder mit Werten zeigt → Tipp ist gesetzt → **Kategorie B, IGNORIEREN**. Auch wenn tippuebersicht "-:-" zeigt (kann verzögert sein).

5. **WebSearch nur für Team-News / Verletzungen** (Phase 3), niemals für Ergebnisse. Auch Team-News-Quellen mit gesundem Misstrauen behandeln.

---

## Phase 1 — kicktipp.de im Browser laden

1. Lade die Chrome-MCP-Tools via ToolSearch:
   `select:mcp__claude-in-chrome__tabs_context_mcp,mcp__claude-in-chrome__navigate,mcp__claude-in-chrome__tabs_create_mcp,mcp__claude-in-chrome__read_page,mcp__claude-in-chrome__computer,mcp__claude-in-chrome__form_input`

2. Prüfe mit `tabs_context_mcp` ob der User bereits eingeloggt ist.

3. Leite die übergebene URL auf die **tippuebersicht** um (ersetze `tippabgabe` durch `tippuebersicht` falls nötig). Öffne in neuem Tab.

4. **Tippuebersicht lesen** (`read_page` + Screenshot):
   - Lies die Ergebnis-Spalte im Header-Table → das sind die **offiziellen Ergebnisse**
   - Lies die BjoernR-Zeile im Spieler-Table → das sind **meine gesetzten Tipps**
   - Notiere für jedes Spiel: Heimteam, Auswärtsteam, Datum, offizielles Ergebnis (oder –), eigener Tipp (oder –)

5. **Tippabgabe lesen** (navigiere zur tippabgabe-URL desselben Spieltags):
   - Spiele mit Eingabefeldern → noch nicht abgeschlossen (offen oder Tipp setzbar)
   - Eingabefeld mit Wert → Tipp bereits gesetzt (auch wenn tippuebersicht noch "-:-" zeigt)
   - Kein Eingabefeld → Spiel abgeschlossen/gesperrt

6. Klassifiziere jedes Spiel:

   | Kategorie | Eigener Tipp | Offizielles Ergebnis | Aktion |
   |-----------|-------------|----------------------|--------|
   | A — Abgeschlossen | ✓ | ✓ | Ergebnis ins Repo schreiben |
   | B — Warten | ✓ | – | **IGNORIEREN** |
   | C — Zu tippen | – | – | Recherche → Tipp → eintippen → Repo |
   | D — Verpasst | – | ✓ | Nur Ergebnis ins Repo schreiben |

---

## Phase 2 — Ergebnisse schreiben (Kategorie A + D)

Für jedes Spiel der Kategorie A oder D:

1. **Bestimme die Spielphase** und die zugehörige Datei:
   - Gruppenphase Runde 1: `spielplan/vorrunde-runde-1.md`
   - Gruppenphase Runde 2: `spielplan/vorrunde-runde-2.md`
   - Gruppenphase Runde 3: `spielplan/vorrunde-runde-3.md`
   - Sechzehntelfinale: `spielplan/sechzehntelfinale.md`
   - Achtelfinale: `spielplan/achtelfinale.md`
   - Viertelfinale: `spielplan/viertelfinale.md`
   - Halbfinale: `spielplan/halbfinale.md`
   - Finale: `spielplan/finale.md`

   Existiert die KO-Datei noch nicht, leg sie nach dem Muster von `sechzehntelfinale.md` an (Header + Tabelle, Spalten: `Datum | Spiel | El Profi Tipp | Ergebnis`).

2. **Finde die Zeile** mit dem Spiel (Heimteam vs. Auswärtsteam).

3. **Trage das Ergebnis** in die Spalte `Ergebnis` ein — **ausschließlich aus der kicktipp-Tippuebersicht**:
   - Normale Zeit: `2:1 BRA`
   - Nach Verlängerung: `1:0 n.V. ESP`
   - Nach Elfmeterschießen: `4:3 n.E. MAR` (Ergebnis nach 120min, Sieger nach Elfmeter)
   - Ersetze `–` durch das konkrete Ergebnis

---

## Phase 3 — Recherche für offene Spiele (Kategorie C)

Für jedes Spiel der Kategorie C — führe **parallel** WebSearch-Abfragen durch (alle Spiele gleichzeitig, nicht nacheinander):

**Suchterme pro Spiel:**
- `[Heimteam] [Auswärtsteam] injury suspension squad 2026`
- `[Heimteam] team news form 2026 World Cup`
- `[Auswärtsteam] Verletzung Sperre WM 2026`

**Auswerten — notiere für jedes Team:**
- Verletzte Stammspieler (Name, Position, Schwere)
- Gesperrte Spieler (Name, Sperre wegen was)
- Aktuelle Form (letzte 3 Spiele im Turnier)
- Besonderheiten (Trainerwechsel, Teamdynamik, Heimvorteil etc.)

**Aktualisiere `news/spieler-events.md`:**
- Füge neue Verletzungen in `## 1. Verletzungen & Ausfälle` ein (Tabelle: Spieler | Team | Verletzung | Details | Tipp-Impact)
- Füge neue Sperren in `## 2. Sperren & Disziplin` ein
- Füge relevante Performances in `## 3. Top-Performances` ein (unter der aktuellen Runde)
- Füge Teamdynamik-Neuigkeiten in `## 4. Teamdynamik & Sonstige News` ein
- Aktualisiere `*Stand: ...*` am Dateianfang auf das heutige Datum

---

## Phase 4 — Tipps generieren (Kategorie C)

Generiere für jedes offene Spiel einen El Profi Tipp basierend auf der Recherche:

**Tipp-Format:**
```
**2:1 ESP** ⚡Mittel
```

**Risikostufen:**
- `⚡Niedrig` — klarer Favorit, keine Überraschungen erwartet
- `⚡Mittel` — Favorit erkennbar, aber Gegner kann mithalten
- `⚡Hoch` — offenes Spiel, beide Teams auf Augenhöhe
- `⚡Sehr hoch` — echter Coin-Flip, Außenseiter hat reale Chance

**KO-Runden-Tipps mit Elfmeter-Erwartung:** Tippe ein konkretes Endergebnis inkl. Elfmeter-Aufschlag (z.B. `3:4 n.E. ARG`, nicht `1:1 n.E. ARG` — kein KO-Spiel endet unentschieden). Erste Zahl = Heimteam, zweite Zahl = Auswärtsteam, in der Reihenfolge wie in der Spielpaarung. Bevorzuge konkrete Sieger.

**Tipp in Spielplan-Datei schreiben:**
Trage den Tipp in die Spalte `El Profi Tipp` der passenden Spielplan-Datei ein. Zeile suchen → Edit-Tool.

---

## Phase 4b — Tipps vom Nutzer bestätigen lassen

Bevor irgend etwas auf kicktipp.de eingetippt wird:

1. Zeige alle generierten Tipps als kompakte Tabelle:

   ```
   | Spiel                        | Tipp          | Risiko     | Begründung (kurz)                     |
   |------------------------------|---------------|------------|---------------------------------------|
   | USA vs. Bosnien              | **2:0 USA**   | ⚡Niedrig  | Heimvorteil, Bosna ohne Džeko         |
   | Spanien vs. Österreich       | **2:1 ESP**   | ⚡Mittel   | Spanien dominant, aber Österreich fit |
   ```

2. Frage explizit: "Soll ich diese Tipps so auf kicktipp.de eintragen? Du kannst einzelne Tipps noch anpassen."

3. **Warte auf Bestätigung des Nutzers.** Erst nach "ja" / "ok" / Korrekturen → Phase 5.

4. Wenn der Nutzer einzelne Tipps ändert: übernimm die Korrekturen sowohl für kicktipp.de (Phase 5) als auch für die Spielplan-Datei.

---

## Phase 5 — Tipps auf kicktipp.de eintippen (Kategorie C)

Für jedes Spiel der Kategorie C — navigiere auf kicktipp.de zur tippabgabe-Seite:

1. Prüfe pro Spiel, ob kicktipp es explizit mit **„n.E."** kennzeichnet (steht direkt neben Datum/Teams, plus Erklärtext: "Bei einem 2:2 nach Verlängerung und einem 5:4 im Elfmeterschießen wird ein 7:6 gewertet."). Wenn ja: **trage das volle erwartete Endergebnis inkl. Elfmeter-Aufschlag ein** (z.B. 3:4), nicht das 90-Minuten-Ergebnis und nicht 1:1. Kein KO-Spiel endet unentschieden — auch der Tipp nicht.
2. Nur wenn ein Spiel NICHT als „n.E." markiert ist (reguläre Gruppenphase o.ä.), das normale 90-Minuten-Ergebnis eintragen.
3. Erstes Eingabefeld = Heimtore, zweites Feld = Auswärtstore — Reihenfolge der Teams auf der Seite beachten (nicht automatisch Favorit zuerst annehmen).
4. Speichern / Absenden
5. Verifiziere per `read_page`/`get_page_text` dass „Die Tipps wurden erfolgreich gespeichert" erscheint
6. Fahre mit nächstem Spiel fort

---

## Phase 6 — Git Commit

Wenn alle Phasen abgeschlossen sind:

1. Zeige eine Zusammenfassung was gemacht wurde:
   - X Ergebnisse eingetragen
   - Y Tipps recherchiert und eingetragen
   - Dateien: [Liste]

2. Committe alle geänderten MD-Dateien:
   ```bash
   git add spielplan/ news/spieler-events.md
   git commit -m "..."
   ```

   Commit-Message-Format:
   - Nur Ergebnisse: `"Sechzehntelfinale: Ergebnisse vom [Datum] eingetragen"`
   - Nur Tipps: `"Sechzehntelfinale: El Profi Tipps für [Datum] gesetzt"`
   - Beides: `"Sechzehntelfinale: Ergebnisse + Tipps für [Datum] aktualisiert"`

---

## Dateiformate (Referenz)

### KO-Runde Spielplan-Header (für neue Dateien)
```markdown
# [Rundenname] — [Datumsbereich]

> Ab der KO-Runde: Verlängerung und Elfmeterschießen möglich.
> Alle Zeiten ET (Eastern Time).

| Datum | Spiel | El Profi Tipp | Ergebnis |
|-------|-------|---------------|----------|
```

### Ergebnis-Notation
- `2:0 USA` — normaler Sieg
- `1:0 n.V. FRA` — Sieg nach Verlängerung
- `4:3 n.E. ARG` — Sieg nach Elfmeterschießen (Ergebnis nach 120min, Sieger nach Elfmeter)

### Tipp-Notation
- `**2:1 ESP** ⚡Mittel` — Gruppenphase und KO (normale Spielzeit)
- `**3:4 n.E. ARG** ⚡Hoch` — wenn Elfmeter erwartet: konkretes Endergebnis inkl. Elfmeter-Aufschlag, NIE `1:1` (kein KO-Spiel endet unentschieden). Reihenfolge Heim:Auswärts.
- Auf kicktipp.de wird bei „n.E."-markierten Spielen exakt dieses Endergebnis eingetragen (nicht das 90-Minuten-Ergebnis) — kicktipp errechnet daraus selbst die Wertung.

### news/spieler-events.md Verletzungs-Tabelle
```
| Spieler | Team | Verletzung | Details | Tipp-Impact |
|---------|------|-----------|---------|-------------|
| Bellingham | England | Oberschenkel | fraglich, Training individuell | Hoch |
```
