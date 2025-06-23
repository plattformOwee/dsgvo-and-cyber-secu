## Ja – eine UG (haftungsbeschränkt) reicht völlig aus

Mit einer UG kannst du schon ab 1 € Stammkapital starten, hast die volle Haftungsbegrenzung einer GmbH und kannst später problemlos auf 25 000 € Stammkapital „hochwachsen“ (25 % Gewinn‐Thesaurierungspflicht!) ([qonto.com](https://qonto.com/de/blog/rechtsformen/ug/gewinnverteilung?utm_source=chatgpt.com "Gewinnverteilung UG (haftungsbeschränkt): Alles Wichtige - Qonto"), [fuer-gruender.de](https://www.fuer-gruender.de/wissen/existenzgruendung-planen/recht-und-steuern/rechtsform/unternehmergesellschaft/?utm_source=chatgpt.com "UG (haftungsbeschränkt) - Vorteile der Mini-GmbH - Für-Gründer.de")).

---

### 1 · Gründungs-To-do (≈ 4–6 Wochen)

|Schritt|Was konkret passiert|Kosten/Tipps|
|---|---|---|
|**1 Namens- & IHK-Check**|Unternehmens­name bei IHK/DPMA prüfen|0 €|
|**2 Gesellschaftervertrag**|Musterprotokoll (1–3 Gesellschafter) **oder** individueller Vertrag|0 – 300 €|
|**3 Notar-Termin**|Beurkundung, HRB-Anmeldung|~300 €|
|**4 Geschäfts­konto & Kapital­einzahlung**|realistisches Start­kapital (≥ 1 000 € empfehlenswert)|–|
|**5 Handelsregister-Eintrag (HRB)**|Haftungs­schutz wird wirksam|~150 € Gerichtskosten|
|**6 Gewerbe­anmeldung**|Rathaus/Wirtschafts­amt|~40 €|
|**7 Fragebogen zur steuerlichen Erfassung** (ELSTER)|Steuernummer + **USt-ID** erhalten|0 €|
|**8 Pflichtmitglied­schaften**|IHK + Berufs­genossenschaft; automatisch|IHK-Grundbeitrag ~ 0–300 € p.a.|

_(Quelle: aktuelle UG-Gründungs­leitfäden 2025) ([gruenderplattform.de](https://gruenderplattform.de/rechtsformen/ug-gruenden?utm_source=chatgpt.com "Eine UG gründen - so geht's Schritt für Schritt - Gründerplattform"), [fuer-gruender.de](https://www.fuer-gruender.de/wissen/unternehmen-gruenden/unternehmensformen/ug-gruenden/?utm_source=chatgpt.com "UG gründen: so geht's schnell, rechtssicher + günstig"), [businesscenter-niederrhein.de](https://businesscenter-niederrhein.de/2025/05/17/ug-haftungsbeschraenkt-gruenden-wichtigste-schritte/?utm_source=chatgpt.com "UG haftungsbeschränkt gründen: Die wichtigsten Schritte im Überblick"))_

---

### 2 · App-Store-Abo varianten & Umsatzsteuer

|Variante|Wer ist „Merchant of Record“?|Wer kümmert sich um USt/VAT?|Was du buchen musst|
|---|---|---|---|
|**A In-App-Purchase (Google/Apple)**|**Google/Apple**|Store zieht länderspezifische VAT ab und führt sie ab. ([support.google.com](https://support.google.com/googleplay/android-developer/answer/138000?hl=en&utm_source=chatgpt.com "Tax rates and value-added tax (VAT) - Play Console Help"), [developer.apple.com](https://developer.apple.com/help/app-store-connect/distributing-apps-in-the-european-union/commissions-fees-and-taxes/?utm_source=chatgpt.com "Commissions, fees, and taxes - App Store Connect - Apple Developer"))|Du buchst _Nettoumsatz_ (= Auszahlung) + Store-Provision als Aufwand.|
|**B Alternativer Zahlungsanbieter** (Paddle, Stripe-Checkout u. a.)|**Du** (UG)|Du musst richtige MwSt aller EU-Kunden erheben → **OSS-Meldung** ab > 10 000 € B2C-EU-Umsatz; ab 2025 existiert ein vereinfachter 100 000 €-Schwellenwert. ([vatcalc.com](https://www.vatcalc.com/eu/eu-vat-on-b2c-digital-services-after-1-july-2021-moss-oss/?utm_source=chatgpt.com "EU VAT on B2C digital services – OSS - vatcalc.com"), [vatcalc.com](https://www.vatcalc.com/eu/eu-2025-vat-registration-thresholds-equivalence-for-foreign-businesses/?utm_source=chatgpt.com "EU 2025 VAT SME Scheme - vatcalc.com"))|Bruttoumsatz + eigene VAT-Abführung; Aufwand = PSP-Gebühr.|

**Praxis-Tipp:** Für den Start genügt Variante A – du hast sofort weltweiten Vertrieb, kein MwSt-Risiko und kannst dich auf Produkt & DSGVO konzentrieren.

---

### 3 · Steuern & Buchhaltung der UG

- **Körperschaftsteuer** 15 % + SolZ (= 15,825 %)
    
- **Gewerbesteuer** je nach Hebesatz (Berlin 14,35 % eff.)
    
- **Umsatzsteuer** 19 % (nur relevant, wenn du selbst Merchant bist)
    
- **Gewinn-Thesaurierung**: 25 % des Jahresüberschusses jährlich in „gesetzliche Rücklage“, bis 25 000 € Stammkapital erreicht sind. ([qonto.com](https://qonto.com/de/blog/rechtsformen/ug/gewinnverteilung?utm_source=chatgpt.com "Gewinnverteilung UG (haftungsbeschränkt): Alles Wichtige - Qonto"))
    

> UG = Kapitalgesellschaft ⇒ **Doppelte Buchführung**, Jahresabschluss (Bilanz + GuV) & Hinterlegung beim Bundesanzeiger. Ein Steuer- oder Buchhaltungs­tool (Lexoffice, DATEV Unternehmen online) ist fast unabdingbar.

---

### 4 · Pflicht-Dokumente für Abo-Geschäft

1. **AGB / ToS** mit Subscription-Klauseln (Preis, Laufzeit, Kündigung, Widerruf § 312 f BGB).
    
2. **DSGVO-konforme Privacy-Policy** (wir haben sie skizziert).
    
3. **DPA/AVV** mit allen Processors (AWS, SendGrid, optional Firebase).
    
4. **Widerrufsformular** + Button „Abo kündigen“ (ein Tap, § 312k BGB).
    
5. **Trader-Info für EU-App-Stores** (Adresse, Kontakt nach DSA). ([developer.apple.com](https://developer.apple.com/help/app-store-connect/manage-compliance-information/manage-european-union-digital-services-act-trader-requirements/?utm_source=chatgpt.com "Manage European Union Digital Services Act trader requirements"))
    

---

### 5 · After-Launch Compliance-Rhythmus

|Zyklus|Aufgabe|
|---|---|
|**Monatlich**|Umsatzsteuer-Voranmeldung (UStVA) oder OSS-Meldung; Abo-KPIs checken|
|**Quartal**|Rücklagenquote prüfen (25 % Regel), IHK-Voraus­zahlung|
|**Jährlich**|Jahresabschluss, E-Bilanz, Offenlegung; DSGVO Review (RoPA, DPO-Pflicht-Check)|
|**On-Demand**|72-h Breach-Process, Data-Export/Delete-Requests|

---

### 6 · Schnell-Checkliste: von „jetzt“ bis ersten Abo-Euro

1. UG gründen (Schritte 1–7 oben)
    
2. Google Play / Apple Dev-Account auf UG umstellen (USt-ID hinterlegen)
    
3. Privacy-Policy & AGB live verlinken, Data-Safety-Form ausfüllen
    
4. Test-Flight / Production-Testing mit In-App-Subscriptions (Sandbox)
    
5. Go-Live → erste Payouts → Buchhaltung einrichten
    
6. Thesaurierung buchen & OSS-Monitoring starten
    

---

**Kurz-Fazit:**  
Du kannst **sofort** mit einer UG starten, die App veröffentlichen und Abo-Revenue einsammeln. Sorge nur für:

- saubere Gründungs­formalitäten,
    
- klaren Umsatzsteuer-Workflow (Store = MoR oder eigener OSS),
    
- Pflicht-Dokus (AGB, Privacy-Policy, AVV) und
    
- eine einfache Buchhaltungs­routine.
    

Damit bist du sowohl gegenüber Investoren als auch Finanzamt & Datenschutz­aufsicht auf der sicheren Seite.



was aav ist:
```
**„AVV“ steht für **Auftrags­verarbeitungs­vertrag – das deutsche Wort für den englischen „Data Processing Agreement (DPA)“ aus Art. 28 DSGVO.  
Wenn du also in meiner Liste „DPA/AVV“ gesehen hast, ist das _ein-und-dasselbe Dokument_, nur in zwei Sprachen ausgedrückt.

### Warum brauchst du einen AVV?

Sobald du einen externen Dienstleister (Processor) einschaltest, der **in deinem Auftrag** personenbezogene Daten verarbeitet (z. B. Hosting-Provider, Mail-Versender, Crash-Reporting-Dienst), bist **du** der „Verantwortliche“ und musst vertraglich festlegen:

1. **welche Daten** der Dienst verarbeitet
    
2. **zu welchem Zweck**
    
3. **welche Sicherheits­maßnahmen** er einhält (TOMs)
    
4. dass er Daten **nur auf deine Weisung** nutzt
    
5. wie er dich bei Auskunfts- oder Lösch­anfragen unterstützt
    
6. seine Pflicht, **Datenpannen sofort** zu melden
    

### Typische Beispiele

|Dienst|AVV vorhanden?|Wo findest du ihn?|
|---|---|---|
|**Hetzner Cloud**|✅|Konsole → „Auftrags­verarbeitung“ (PDF)|
|**AWS (eu-central-1)**|✅|„AWS Data Processing Addendum“ wird online akzeptiert|
|**SendGrid**|✅|Teil der Terms & Conditions („Data Processing Addendum + SCC“)|
|**Firebase Crashlytics**|✅|Console → Account > Data Processing and Security Terms|

### Muss ich selbst einen Vertrag schreiben?

Meist nicht. Große Anbieter liefern einen **vorgefertigten AVV**: Du klickst „Accept“ oder lädst das PDF herunter.  
Bei kleinen EU-Dienstleistern musst du eventuell ein **eigenes Muster** schicken oder heften beide Seiten eine Standard-Vorlage an.

> **Aufbewahrung:** Lade das unterzeichnete PDF herunter und lege es in deinen _compliance/legal_-Ordner. Auditoren oder Investoren wollen bei Bedarf alle AVVs sehen.

### Kurz-Fazit

- „AVV“ = deutsches Synonym für „DPA“.
    
- Ohne AVV mit jedem Auftrags­verarbeiter verletzt du Art. 28 DSGVO.
    
- In der Praxis genügt oft ein Klick in der Anbieter-Konsole – aber bewahre die Datei als Nachweis auf.
```
