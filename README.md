# Solar-Batterie 5 V (USB-C)

Selbstbau-Solarbatterie mit 2S3P-18650-Akkupack, MPPT-Laderegler und 5-V-USB-C-Ausgang. Das Projekt ist aufgebaut, getestet und funktionsfähig (Stand 27.09.2026).

Autor: Paul Simmen mit Vibe (GLM-5-latest-short), 27.09.2026

## Übersicht

| Baugruppe | Komponente | Funktion |
|---|---|---|
| Panel | Solarmodul, U > 18 V, Imax 2000 mA | Energiequelle (Voc ca. 21–22 V) |
| Laderegler | ODLKJ V09A («MPPT») | lädt den Akkupack (Buck-Lader) |
| Akkupack | 6× 18650 à 3300 mAh, 2S3P | Speicher, ca. 8,4 V / ~9,9 Ah |
| BMS | HX-2S JH20 V1.0 | Schutz + Balancer (2S, Common-Port) |
| DC-DC-Wandler | LM2596S | erzeugt 5 V Ausgangsspannung |
| Ausgang | USB-C-Socken | Versorgung von USB-Geräten |

## Funktionsprinzip

Das Panel speist den MPPT-Lader. Dieser lädt den 2S3P-Akkupack (Ladeschlussspannung 8,4 V) über den Port des BMS. Der LM2596S wandelt die Pack-Spannung (8,4–7,0 V) auf stabile 5 V und versorgt den USB-C-Ausgang. Lader und Wandler hängen beide am Common-Port P+/P– des BMS – so schützt das BMS den Pack in beide Richtungen (Überladung, Tiefentladung, Kurzschluss, Balance).

```
Solar-Panel (18 V)
      │
      ▼
MPPT-Lader ODLKJ V09A ──► P+ / P– (BMS HX-2S JH20)
                              │
                       6× 18650 (2S3P)
                       mit Balancer (BM)
                              │
                      P+ / P– ──► LM2596S ──► USB-C (5 V)
```

## Verdrahtung

### Anschlüsse des BMS HX-2S JH20

Das Board ist ein **Common-Port-BMS**: Laden und Entladen laufen über dieselben P-Anschlüsse. Es gibt einen separaten Balance-Draht (BM).

| Anschluss | Wohin |
|---|---|
| B+ | Plus des Packs (Zellgruppe 1) |
| BM | Mittelpunkt zwischen den beiden Zellgruppen (Balancer – zwingend anzuschliessen) |
| B– | Minus des Packs (Zellgruppe 2) |
| P+ / P– | MPPT-Lader **und** LM2596 gemeinsam |

### Kritische Anschlussreihenfolge

Beim ersten Anklemmen des BMS zwingend einhalten:

1. Zellen zuerst fertig zum 2S3P-Pack verschalten
2. **B–** anklemmen
3. **BM** anklemmen
4. **B+** anklemmen

In anderer Reihenfolge können über die Balancer-Endstufe Ausgleichsströme fliessen und die Balancer-FETs zerstört werden.

### Vorher zwingend prüfen

- **Beide Zellgruppen auf nahezu gleiche Spannung bringen** (< 0,05 V Differenz, ideal: alle 6 Zellen einzeln auf 4,0–4,1 V laden). Sonst fliessen beim Zusammenschalten unkontrollierte Ausgleichsströme zwischen den Zellen.
- Keine alten und neuen Zellen mischen.
- Spannung an P+/P– messen (manche Boards schalten erst nach kurzem Antippen mit einer Last frei).

## Einstellungen

### Solarregler ODLKJ V09A (zwei Poti)

| Poti | Funktion | Einstellung |
|---|---|---|
| U-Out | Ladeschlussspannung (Akkuseite) | **8,4 V** (für 2S Li-Ion) |
| U-MPPT | Panel-Arbeitspunkt | **ca. 17–18 V** (≈ MPP-Spannung des 18-V-Panels, alternativ ~0,8 × Voc) |

**U-MPPT live einstellen:** Bei voller Sonne Multimeter an den Regler-Eingang (Panel +/–), Poti drehen, bis die Panelspannung unter Last auf dem Sollwert «klebt» und der Ladestrom maximal ist. Zu hoch gedreht → Panelspannung bricht ein, Ladestrom sinkt; zu tief → verschenkte Panelleistung.

**Hinweis:** Viele dieser «MPPT»-Boards sind keine echten MPP-Tracker, sondern einfache Buck-Lader mit Eingangsspannungs-Regelung – für den Zweck des Ladens ausreichend.

## Ladezustand vs. Spannung (2S-Pack, Ruhespannung)

Werte gelten ohne Last, einige Minuten nach dem Laden/Entladen. Prozentwerte sind Näherungen – die reale Kurve ist flach und hängt von Zellentyp, Strom und Temperatur ab.

| Ladung % | Zellenspannung | Pack-Spannung (2S) | Bemerkung |
|---|---|---|---|
| **100 %** | 4,20 V | 8,40 V | Ladeschluss (U-Out des Reglers) |
| 90 % | 4,08 V | 8,16 V | |
| 80 % | 4,00 V | 8,00 V | Alltagsbereich beginnt |
| 70 % | 3,93 V | 7,86 V | |
| 60 % | 3,87 V | 7,74 V | |
| 50 % | 3,82 V | 7,64 V | mittlerer Arbeitsbereich |
| 40 % | 3,79 V | 7,58 V | |
| 30 % | 3,75 V | 7,50 V | |
| 20 % | 3,70 V | 7,40 V | Nennspannung des Packs |
| 10 % | 3,60 V | 7,20 V | ⚠️ LM2596 wird knapp (braucht ~7 V Eingang) |
| 5 % | 3,50 V | 7,00 V | Grenze für stabile 5 V |
| **0 %** | 3,00 V | 6,00 V | Tiefentladeschutz des JH20 schaltet ab |

**Konsequenz:** Der praktisch nutzbare Bereich endet bei ca. 7,2 V (~10 % Ladezustand). Darunter hält der LM2596 die 5 V nicht mehr stabil, obwohl das BMS erst bei 6,0 V abschaltet. Unter Last (0,05–0,1 V pro Zelle Innenwiderstand) zeigt das Multimeter zusätzlich zu tief – verlässliche Werte gibt es nach ~30 Min. Ruhe, gemessen zwischen B+ und B–.

## Testergebnisse (27.09.2026)

- Aufbau fertiggestellt, alle Zellen voll geladen eingesetzt: **Funktion bestätigt** ✔️
- Testlast: 4× 10 Ω in Serie (40 Ω) → 5 V / 40 Ω = **125 mA** (0,625 W) am Ausgang – Schonbetrieb
- Bei dieser Last ist der Eigenverbrauch von BMS + LM2596 (geschätzt 5–20 mA) relevant; beim Handyladen (1–2 A) ist er vernachlässigbar
- Erwartung fürs Handyladen: LM2596 wird mit 2–3 W Verlustleistung deutlich warm (60–70 °C noch normal), darüber Kühlblech; realistische Dauerlast max. 2 A

## Fallstricke (Lessons Learned)

1. **Zellen angleichen** vor dem Parallelschalten – gleiche Spannung, kein Mischbestand aus alt/neu.
2. **Lader und Last korrekt an die BMS-Ports**: beide an P+/P–; niemals Last direkt an B+/B– abgreifen (sonst kein Tiefentladungs-/Kurzschlussschutz); BM immer anschliessen.
3. **USB-C-Socken braucht 5,1-kΩ-Widerstände** von CC1/CC nach GND (Rd-Pull-Downs) – sonst verhandelt kein modernes Gerät und lädt nicht.
4. LM2596 braucht ca. 2 V Differenz: unter ~7 V Pack-Spannung fällt die 5-V-Schiene ab → nutzbarer SoC-Bereich endet bei ~10 % (Alternative: Buck-Boost-Wandler).
5. BMS und LM2596 haben Ruhestrom (je einige mA) → Pack leert ohne Last langsam; bei Standby relevant.
6. Sicherung (3–5 A) zwischen Pack und LM2596 einplanen; nicht unter 0 °C laden; Pack beim Laden nicht >45 °C.
7. 18650 möglichst punktschweissen (Ni-Streifen); Federkontakte für Dauerstrom grenzwertig.
8. Vorsicht bei Lasttests mit Widerständen: 4× 10 Ω parallel = 2,5 Ω → je Widerstand 2,5 W Verlustleistung; Standardwiderstände (0,25–1 W) überleben das nicht.

## Offene Punkte / nächste Schritte

- [ ] 5 V unter Last nachmessen (Soll: 5,0 V ± 0,25 V)
- [ ] U-MPPT bei voller Sonne auf ~17–18 V einstellen
- [ ] USB-C-Socken mit 5,1-kΩ-CC-Pulldowns einbauen
- [ ] Beide Zellgruppen nach Test einzeln messen (Balancer-Kontrolle, Differenz < 0,05 V)
- [ ] Ruhestrom ohne Last messen (wie schnell leert der Pack im Standby?)
- [ ] Verhalten nach Tiefentladung prüfen (schaltet das JH20 den Ladestrom mit ab?)
- [ ] Lasttest mit 1–2 A (Handy oder geeignete Leistungswiderstände)

## Quellen & Verweise

- Projektdokumentation im KI-Notizbuch (Google Drive), Ordner 04_Projekte: *2026-09-27_Solarbatterie-USB-C*
- Begleitende Diskussion und Dimensionierung: Vibe-Projekt «Basteleien» (27.09.2026)

---

Autor: Paul Simmen mit Vibe (GLM-5-latest-short), 27.09.2026
