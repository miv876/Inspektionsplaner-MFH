# Inspektionsplaner MFH

Terminierung von Wohnungsinspektionen in Mehrfamilienhäusern (Terradata AG).
Eine einzelne HTML-Datei ohne Build und ohne Server: `index.html` im Browser öffnen.

## Funktionen

- **Import** des GeoJSON-Exports aus dem Offert-Perimeter-Tool (mehrere Dateien werden zusammengeführt)
- **Wohnungen aus dem GWR** (geo.admin.ch) per EGID, alternativ per Adresse; manuell erfassen und bearbeiten
- **Terminplan je Adresse** nach Stockwerk und Wohnung, mit allgemeinen Räumen und Treppenhaus
- **Parameter** mit Standardwerten: Arbeitszeit, Mittagspause, Arbeitstage, Dauern, Reihenfolge, Wechselzeiten
- **Einsatztage** als Serie aufeinanderfolgender Arbeitstage, jedes Datum einzeln übersteuerbar
- **Übersicht** mit Kennzahlen, Adressliste, kalendertreuem Gantt (Kalender/Kompakt) und allen Terminen
- **Karte** (Leaflet, swisstopo/AV) mit Perimeter-Objekten, nummerierten Adressen und «+ aus Karte» (GWR-Abfrage am Klickpunkt)
- **Halbtags-Einsätze** (VM/NM) mit Team-Zuteilung, `.ics`-Kalenderdateien und Einladungen (Outlook-Entwurf `.eml`, mailto, Text kopieren)
- **Serienbrief** an alle Wohnungen: Empfänger je Wohnung, Version A/B/C je Wohnung (Textbaustein, Betreff, Bild), Vorschau, Word-Serienbrief als eine `.docx`, Einzelbriefe als ZIP, Seriendruck-Daten als Excel
- **Export** Excel (Übersicht mit Kartenbild, Einsätze, Gantt, alle Termine, Serienbrief-Daten, ein Blatt je Adresse; Logo im Kopf) und Projekt-JSON (wieder ladbar)
- Hell/Dunkel-Modus, eigenes Logo, Beispieldaten ein-/ausblendbar

## Standardwerte

| Parameter | Standard |
|---|---|
| Arbeitszeit | 08:00–17:00, Mo–Fr |
| Mittagspause | 12:00–13:00 |
| Wohnung gross (ab 3 Zimmer) inkl. Keller | 30 min |
| Wohnung klein inkl. Keller | 15 min |
| Allgemeine Räume je Adresse | 30 min |
| Treppenhaus | 5 min je Geschoss |
| Wechselzeit / Adresswechsel | 5 min / 15 min |

## Eingabeformat (GeoJSON)

Erwartet wird eine `FeatureCollection` (CRS84). Je Feature werden die Attribute unter
`properties.attribute` gelesen, verschachtelt (`gebaeude`, `grundstueck`) oder flach:

```json
"attribute": {
  "gebaeude": {
    "Gebäudeadresse": "Beispielweg 4, 8304 Wallisellen",
    "EGID": "123456789",
    "Anzahl Geschosse": 4,
    "Anzahl Wohnungen": 7
  }
}
```

Features ohne Gebäudeadresse werden als Kartenobjekte angezeigt, aber nicht geplant.
Mehrere Features mit gleicher EGID ergeben eine Adresse. Ohne GWR-Abruf werden
Platzhalter-Wohnungen aus «Anzahl Wohnungen» erzeugt. Ein fiktives Beispiel liegt in
`examples/beispiel_perimeter.geojson`.

## Serienbrief

Tabs **Serienbrief** (Empfänger je Wohnung, Version wählen, Briefe erzeugen) und **Briefvorlagen**
(Absender, Zeitfenster, Versionen A/B/C mit Betreff, Textbaustein und Bild, Word-Vorlage).

- **Empfänger** sind an der Wohnung gespeichert. Ohne Namen geht der Brief an «An die Bewohnerinnen
  und Bewohner, Wohnung …» an die Adresse der Liegenschaft; eine abweichende Postadresse ist möglich.
- **Import** über «Empfänger importieren» als JSON, Excel (`.xlsx`) oder CSV. Erkannte Spalten:
  `Adresse` oder `EGID`, `Wohnung` oder `EWID`, `Anrede`, `Vorname`, `Name`, `Zusatz`, `Strasse`,
  `PLZ`, `Ort`, `Version` (A/B/C), `Brief` (ja/nein). Zuordnung: EGID bzw. Adresse (Strasse und Nr.)
  und Wohnungsnummer bzw. EWID.
- **Export** «Empfänger exportieren (JSON)» im Format `inspektionsplaner-empfaenger`; ein Re-Import
  ist über denselben Knopf oder «Import» möglich.
- **Projekt** «Projekt (JSON)» speichert alles (Adressen, Wohnungen, Empfänger, Versionen inkl. Bilder,
  Word-Vorlage, Team, Einsatztage); «Import» lädt es wieder.

### Word-Vorlage

Ohne eigene Vorlage verwendet die App eine eingebaute Standardvorlage (A4, Adressfeld rechts für
Fenstercouvert, Logo). Sie lässt sich unter «Briefvorlagen» herunterladen und in Word anpassen.

Eine eigene `.docx` beschreibt **einen** Brief. Platzhalter in geschweiften Klammern, z. B.
`{empfaenger}`, `{betreff}`, `{briefanrede}`, `{text}`, `{termin_datum}`. Inhalte nur für eine Version
(Text, Bilder) stehen zwischen `{#varA}` und `{/varA}` (analog B, C). Die App fasst den Inhalt für den
Serienbrief selbst in eine Schleife mit Seitenumbruch. Kopf- und Fusszeile sollten nur feste Inhalte
enthalten. Erzeugung im Browser mit [docxtemplater](https://docxtemplater.com) und PizZip.

| Platzhalter | Inhalt |
|---|---|
| `empfaenger` | Adressblock, mehrzeilig |
| `absender`, `abs_firma`, `abs_zeile2`, `abs_zeile3` | Absender |
| `ort_datum` | Ort, Datum der Erstellung |
| `betreff`, `text` | Betreff und Textbaustein der gewählten Version (Platzhalter darin bereits ersetzt) |
| `briefanrede` | z. B. «Sehr geehrte Frau Muster», sonst «Sehr geehrte Damen und Herren» |
| `adresse`, `liegenschaft`, `wohnung`, `stockwerk`, `lage`, `zimmer` | Objekt |
| `termin_wochentag`, `termin_datum`, `termin_von`, `termin_bis`, `termin_fenster`, `dauer` | Termin |
| `anrede`, `vorname`, `name`, `zusatz`, `post_strasse`, `post_plzort` | Empfänger |
| `unterzeichner`, `funktion`, `version` | Weitere |

## Externe Dienste

| Dienst | Zweck |
|---|---|
| `api3.geo.admin.ch` | GWR-Abfragen (find, identify, Feature) |
| `wmts.geo.admin.ch`, `wms.geo.admin.ch` | Hintergrundkarten, Kartenbild für Excel |
| `cdnjs.cloudflare.com` | ExcelJS 4.3.0, Leaflet 1.9.4 |
| `cdn.jsdelivr.net` | PizZip 3.3.0, docxtemplater 3.71.1 (Word-Erzeugung) |
| `fonts.googleapis.com` | Schriften (Archivo, Source Sans 3, JetBrains Mono) |

Ohne Internet funktionieren Planung und JSON-Export; Excel, Karte und GWR nicht.

## Hinweise

- Logo, Team, Organisator, Briefvorlagen, Hell/Dunkel und Beispieldaten-Schalter werden im `localStorage` des Browsers gespeichert. Plan und Empfänger nicht: für die Ablage «Projekt (JSON)» verwenden.
- `.eml`-Entwürfe enthalten `X-Unsent: 1`. Outlook-Desktop (Windows) öffnet sie als sendebereiten Entwurf; andere Programme zeigen sie als empfangene Mail.
- Kalendereinladungen mit Zu-/Absage (`METHOD:REQUEST`) entstehen nur mit Organisator-E-Mail; sonst einfache Einträge (`METHOD:PUBLISH`).
- In der claude.ai-Artifact-Ansicht sind externe Abfragen gesperrt und `.ics`/`.eml` kommen als ZIP. Lokal geöffnet entfällt beides.
