### Kurz & knackig — so hängen **Rechts­grund­lage**-Mapping und **RoPA** zusammen

|Zweck|Was steht drin?|Wer braucht’s wofür?|Wie nutzt du es praktisch?|
|---|---|---|---|
|**1. „Rechtsgrundlage festlegen“**(Legal-Basis-Mapping, Art. 6 / Art. 9 DSGVO)|_Jeder_ einzelne Verarbeitungs­vorgang → welche Datenfelder → **auf welche Rechts­grundlage** stützt du dich? (Vertrag, Einwilligung, berechtigtes Interesse …)|- Produkt- & Dev-Team ⇒ entscheidet, ob ein Opt-in-Dialog nötig ist - UX / Copy-Writer ⇒ Text für Checkbox, Hint, Balancing-Test - Anwalt ⇒ prüft, ob Auswahl schlüssig|_Vor_ dem Codieren: 1. Tabelle ausfüllen (dein Sheet „Praxis-Mapping Art. 6 / 9“). 2. Daraus: UI-Flows, Consent-Flags, Privacy-Policy-Abschnitte erzeugen.|
|**2. RoPA** (Record of Processing Activities, Art. 30 DSGVO – „Verarbeitungs­verzeichnis“)|Gesamt­übersicht pro **Daten­kategorie**: Zweck, Rechts­grundlage, Empfänger, Aufbewahrungs­frist, TOMs (Security-Maßnahmen)|- Datenschutz­behörde (kann es innerhalb = 30 Tagen verlangen) - Interne/r DSB / Gründer für Audits|_Nach_ der Fach-Entscheidung: 1. Zeile je Daten­gruppe (siehe deine große Tabelle). 2. Ablegen als PDF/Sheet; jedes Jahr oder bei Schema-Änderung aktualisieren.|

**Merke:**  
_Das Rechts­grundlagen-Mapping ist die **Vorlauf-Analyse**,_ aus der du deine Opt-in-Flows und Policy-Texte baust.  
_Die RoPA ist das **amtliche Register**,_ in dem dieselben Infos – aber verdichtet nach Kategorien + mit Aufbewahrungsfristen + TOMs – für Behörden hinterlegt werden.

---

## So gehst du vor

1. **Fülle / aktualisiere** dein **Legal-Basis-Sheet** (Spalte „WIE du sie erfüllst“ entscheidet, ob Checkbox, Double-Opt-In usw.).
    
2. **Implementiere** die abgeleiteten Punkte in Code & UI (Consent-Dialog, Balancing-Test-PDF bei lit. f, Newsletter-Opt-In …).
    
3. **Überführe** die finalen Spalten „Zweck | Rechtsgrundlage | Löschfrist | TOM“ pro Datenblock in dein **RoPA-Dokument** (kann ein zweiter Tab sein).
    
4. **Versionieren & signieren** – jedes Mal, wenn du das DB-Schema änderst, Legal-Basis und RoPA zusammen updaten (Git-Commit + „v2025-07-xx“).
    

So hast du erst die **Entscheidungs­grundlage** (Legal-Basis-Mapping) und daraus das **Behörden­taugliche Register** (RoPA) — beides nötig, aber mit klar getrenntem Zweck.