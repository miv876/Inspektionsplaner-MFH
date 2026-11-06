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
- **Export** Excel (Übersicht mit Kartenbild, Einsätze, Gantt, alle Termine, ein Blatt je Adresse; Logo im Kopf) und JSON
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

## Externe Dienste

| Dienst | Zweck |
|---|---|
| `api3.geo.admin.ch` | GWR-Abfragen (find, identify, Feature) |
| `wmts.geo.admin.ch`, `wms.geo.admin.ch` | Hintergrundkarten, Kartenbild für Excel |
| `cdnjs.cloudflare.com` | ExcelJS 4.3.0, Leaflet 1.9.4 |
| `fonts.googleapis.com` | Schriften (Archivo, Source Sans 3, JetBrains Mono) |

Ohne Internet funktionieren Planung und JSON-Export; Excel, Karte und GWR nicht.

## Hinweise

- Logo, Team, Organisator, Hell/Dunkel und Beispieldaten-Schalter werden im `localStorage` des Browsers gespeichert. Der Plan selbst nicht: für die Ablage den JSON-Export verwenden.
- `.eml`-Entwürfe enthalten `X-Unsent: 1`. Outlook-Desktop (Windows) öffnet sie als sendebereiten Entwurf; andere Programme zeigen sie als empfangene Mail.
- Kalendereinladungen mit Zu-/Absage (`METHOD:REQUEST`) entstehen nur mit Organisator-E-Mail; sonst einfache Einträge (`METHOD:PUBLISH`).
- In der claude.ai-Artifact-Ansicht sind externe Abfragen gesperrt und `.ics`/`.eml` kommen als ZIP. Lokal geöffnet entfällt beides.
