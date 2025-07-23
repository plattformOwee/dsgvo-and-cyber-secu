- [ ] [[DSGVO ]] [[DSGVO TODO]] [[DSGVO TODO LIST]]
	- [ ] [[RoPA woven Verarbeitungsverzeichnis]] > dokument das wo liegen und auf anfrage bereit gestellt werden muss und updated werden muss bei jeglichen Änderungen
		- [ ] checken ob alle dinge die hier mit einwilligung begründed sind wirklich in den AGBs ist 
		- [ ] checken ob die RoPA wirklich alle felder die es gibt nennt
	- [ ] Rechtsgrundlage festlegen Art. 6 DSGVO “Vertrag” (Nutzungs-Abo) reicht meistens; für optionale Mails/Analytics zusätzliche Einwilligung" [[Rechtsgrundlage festlegen]] > als json dokument iwo 
	- [ ] [[Privacy Policy und Impressum]]
		- [ ] Your privacy notice (or “how to exercise your rights” page) must explain how a user can request restriction—e.g. “Please e-mail privacy@yourdomain.com with ‘Request restriction’ in the subject.”
			- [ ] probably muss es auch in request_me stehen aber das nochmal researchen
	- [ ] [[crashlytics]]
	- [ ] [[User-Rights-API]]
		- [x] delete_me.php
			- [x] script exists and works by recording the request
			- [x] + a script to look each night for users to delete cron job
				- [x] sudo nano /var/www/woven/scripts/cleanup_delete_requests.php
				- [x] sudo tail -n 20 /var/www/woven/logs/cleanup.log
			- [x] re-auth prompt also entweder nochmal nach passwort oder nach device auth fragen
		- [x] request_me.php
			- [x] umgeschrieben benutzt kein aws s3 mehr
			- [x] cron job gemacht der die page jeweils nach 24 stunden immer löscht cleanup_exports.sh 
			- [x] re-auth prompt also entweder nochmal nach passwort oder nach device auth fragen
				- [x] mal passwort in login und signup flow integrieren lul
			- [x] checken ob hierin alle "export page" nötigen dinge drin sind (#3)
		- [x] Rectification
			- [ ] eine hinter authentifizierung geschützte seite wo der nutzer alle seine daten angleichen kann mit sofortiger wirkung
		- [x] Objection
			- [ ] nutzer kann alle persmissions jederzeit ändern nach vorheriger authentifizierung in "permissions page"
		- [ ] Restriction
			- [ ] das soll es geben als möglichkeit während andere dinge ablaufen die aber hier immer instant passieren aktuell, 
				- [ ] mal fragen, ob das restriction recht ausreichend bedient ist, wenn man es iwie formel per email anfragen kann oder ob es easily accessable usw sein muss
				- [ ] egal > wir geben die möglichkeit einfach durchgehend dann muss man sich darum nicht kümmern
			- [ ] restrict and unrestrict in page implementiert
			- [ ] auth abrfage vorher
			- [ ] [[todo für chatgpt]]
			- [ ] alle scripte dementsprechend reagieren lassen
				- [ ] was muss passieren / gelassen werden bei restricted processing
		- [ ] von #5 privacy policy abschnitt schauen, dass er wirklich in der privacy policy is
		- [ ] vojn #4 schauen das die sicherheit gewährt ist
			- [ ] rate limit (DoS)
			- [ ] re auth prompt
			- [ ] download url nur 24 h gültig
			- [ ] **2-Man-Rule für Admin-Löschung**     |      Wenn Ops manuell eingreifen muss    kp nochnal chatgpt fragen 
	- [ ] [[Datenminimierung & Security]]
	- [ ] [[Auftragsverarbeitung]]
	- [ ] [[Breach-Process]]
	- [ ] [[unterschied zwischen RoPa und rechtsgrundlage festlegen]]
	- [ ] [[encrypt as much as possible]]
	- [ ] [[chatgpt fragen was noch fehlt]]

- [x] Geld für notar auftreiben 7
	- [ ] herausfinden wv es ist
		- [ ] email geschrieben
	- [x] es auftreiben von lukas


- [ ] [[cyber security]]
	- [x] http > https
	- [ ] encryption
		- [ ] encrypt in transit TLS SSL
			- [ ] prüfen ob es bei allen services der fall ist
				- [ ] mongodb
				- [ ] php
				- [ ] https idk
		- [ ] encrypt at rest
			- [ ] find out what that even mean


- [ ] y-combinator bewerbung
	- [ ] braune haarfarbe kaufen haare anders machen mit hellen spitzen 
	- [ ] linked-in profil machen
	- [ ] bewerben


- [ ] Geld
	- [ ] von gerd behalten
	- [ ] von milan bekommen
		- [ ] ihn endlich erreichen


- [ ] firmenanmeldung und rechtliches
	- [x] geld
	- [x] beglaubigt.de
	- [x] [[Gründungsurkunde und Gesellschaftsvertrag]]
	- [x] notar termin
	- [x] geld dafür 
	- [ ] create geschäftkonto
		- [ ] research the geschäftskonto market 
		- [ ] decide for one
		- [ ] make oppointment
		- [ ] get money from lukas
		- [ ] got there create and pay in
	- [ ] send confirmation to notary and request to be listen in handelsregister
	- [ ] ins transparenzregister eintragen
		- [ ] herausfinden ob das vor nach handelsregister passieren soll
			- [ ] anrufen bei beglaubigt
			- [ ] beim nächsten notar termin nachfragen
	- [ ] firmen namen auf das Klingelschild packen weil geht nicht nach namen
	- [ ] beim finanzamt melden und elster account einrichten
	- [ ] herausfinden wie abbo Einnahmen zu verbuchen sind
		- [ ] anfangs manuell
		- [ ] dann schauen wie man es automatisieren kann




- [ ] Marke sichern
	- [ ] [[bestehende woven marke revoken]]
	- [ ] voven sichern


- [ ] App
	- [ ] weniger weiß
		- [ ] dickere schrift
	- [ ] unterschiedliche bild größen  und bilder verteilt durchs profiel
	- [x] ChooseMethodScreen > AGB's und Impressum hinzufügen (gleiches bei login version)


- [x] neuen signup flow
- [x] xd signup flow designen mit erklärungen und so dazwischen
- [x] das caption über fotos design problem:
	- [ ] einen screen nach dem zuschneiden einfügen in dem man das bild element sieht und ne caption hinzufügen kann
	- [x] erstmal einfach image carousel haben weil sonst muss design voll geändert werden eig






- [ ] was brauch ich von wem um welchem dieser ziele zu dienen und wie bekomme ich es SCHNELLSTMÖGLICH?
	- [x] firmenanmeldung > geld > 
		- [x] lukas /
		- [x] gerd [[nachricht an gerd]]
		- [ ] ~~paps? (zumindest mal fragen könnte man)~~
	- [ ] besseres design
		- [ ] betty (geld ) >
			- [ ] gerd
	- [ ] bestätigung über dsgvo anforderungen und cyber security anforderungen
		- [x] digital mountain
		- [ ] andere beratungsfirmen vielleicht
		- [x] holger
			- [ ] carsten
			- [ ] daniel
	- [x] server wieder laufen haben
		- [x]  felix motivieren oder einladen
		- [x] eigenen kaufen / geld >
			- [x] rita schreiben
			- [ ] 







# wenn noch zeit ist
- [ ] App Design
	- [ ] [[more beautiful edit profile info page]]



# todo now 1
- [ ] research the geschäftskonto market 
- [ ] decide for one
- [ ] make oppointment
- [ ] dsvgo object rectify and so on
	- [ ] 
- [ ] 




- [ ] create company
	- [ ] create geschäftkonto
		- [ ] research the geschäftskonto market 
		- [ ] decide for one
		- [ ] make oppointment
		- [ ] get money from lukas
		- [ ] go there create and pay in
	- [ ] send confirmation to notary and request to be listen in handelsregister
	- [ ] ins transparenzregister eintragen
		- [ ] herausfinden ob das vor nach handelsregister passieren soll
			- [ ] anrufen bei beglaubigt
			- [ ] beim nächsten notar termin nachfragen
	- [ ] firmen namen auf das Klingelschild packen weil geht nicht nach namen




































##### To-do-Liste – Vorbereitung & Einreichung eines Nichtbenutzungsantrags (Art. 58 EUTMR) gegen EUTM 017995606 „WOVEN“

- [ ] **Daten & Fristen bestätigen**
  - [ ] Eintrag in EUIPO eSearch Plus öffnen und Screenshots der Seiten „Timeline“ und „Goods & Services“ machen (inkl. Datumstempel)
  - [ ] Ablauffrist der Benutzungsschonfrist (04 Aug 2024) sowie Erneuerungsfrist (04 Dez 2028) in Kalender/Reminder eintragen

- [ ] **EUIPO-Aktenbestand herunterladen**
  - [ ] In Reiter **Documents** alle PDFs unter *Incoming*, *Outgoing*, *Attachments* speichern
  - [ ] Prüfen, ob jemals Benutzungsnachweise (proof of use) eingereicht wurden
  - [ ] Prüfen, ob außer **C 2.2** weitere Recordals (Assignment C 1.1, License C 4.x) existieren

- [ ] **Öffentliche Quellen auf Benutzung durchsuchen (08 / 2019 – 08 / 2024)**
  - [ ] **App Store (iOS)**
    - [ ] Region jeweils auf 🇩🇪 DE, 🇫🇷 FR, 🇪🇸 ES etc. umstellen
    - [ ] Suche nach „Woven ID“ / Publisher „Centinal“ – Screenshot mit Versionshistorie
  - [ ] **Google Play Store**
    - [ ] Gleiches Vorgehen wie iOS, EU-Regionen via VPN simulieren
  - [ ] **Wayback Machine / Web-Archiv**
    - [ ] Domains wie `wovensolutions.io`, `centinal.com`, `wovenid.com` eingeben
    - [ ] Snapshots 2019-2024 mit **EU-Bezug** (€, GDPR-Cookie-Banner etc.) als PDF sichern
  - [ ] **Web-Suche (Google, Bing, DuckDuckGo)**
    - [ ] `"Woven" AND "Centinal" AND 2020` usw. – News- und Blog-Posts sammeln
  - [ ] **Social Media & PR**
    - [ ] LinkedIn-Firmenprofil von Centinal, LLC → Standortfilter „Europe“
    - [ ] Twitter/X-Suche nach `from:Centinal since:2019-08-04 until:2024-08-04`
  - [ ] **Datenbanken & Register**
    - [ ] Crunchbase/PitchBook auf Finanzmeldungen mit EU-Kunden prüfen
    - [ ] Nationale Register (Companies House, Danish CVR, etc.) auf Zweigniederlassungen durchsuchen

- [ ] **Beweismittel filtern ⇒ „PTEN-Check“**
  - [ ] **P**lace – Sind die Nachweise klar EU-bezogen?
  - [ ] **T**ime – Liegen sie innerhalb 04 Aug 2019 – 04 Aug 2024?
  - [ ] **E**xtent – Deutlich mehr als rein „token“ (z. B. > 100 Downloads, echte Umsätze)?
  - [ ] **N**ature – Betreffen sie Software-/SaaS-/ID-Dienste gem. Klassen 9/42/45?

- [ ] **PDF-Dossier „Kein Nachweis ernsthafter Nutzung“ erstellen**
  - [ ] Deckblatt mit Aktenzeichen, Datum, Antragsteller
  - [ ] Tabellarische Übersicht der durchsuchten Quellen & Ergebnis („no hit“ / Screenshot)
  - [ ] Anhang: alle Screenshots & Archive-PDFs, jeweils mit URL, Datum, Zeitstempel
  - [ ] Als eine einzige durchsuchbare PDF (OCR) abspeichern

- [ ] **Nichtbenutzungsantrag vorbereiten**
  - [ ] EUIPO-Benutzerkonto („User Area“) anlegen
  - [ ] eCancellation-Formular öffnen → „Revocation – Non-use“
  - [ ] Antragsfeld ausfüllen:
    - [ ] Markennummer, Klassen 9 / 42 / 45
    - [ ] Zeitfenster: 04 Aug 2019 – 04 Aug 2024
    - [ ] Antrag auf vollständige/partielle Löschung (nach Ergebnis PTEN-Check)
  - [ ] PDF-Dossier hochladen
  - [ ] Gebühr € 630 online bezahlen

- [ ] **Nach Einreichung Fristen überwachen**
  - [ ] 3-Monats-Frist + evtl. 2-Monats-Verlängerung für Gegenseite in Kalender setzen
  - [ ] EUIPO-Mitteilungen (MyPage > Inbox) wöchentlich prüfen
  - [ ] Falls Beweismittel vom Inhaber eingehen:
    - [ ] Bewertung → ggf. Stellungnahme („Observations on Evidence“) binnen 1 Monat einreichen

- [ ] **Kostenvorsorge / Budget**
  - [ ] Rücklage € 500 für eventuelle Kostenerstattung an Gegner bei Unterliegen
  - [ ] Optionale Anwaltsprüfung des Dossiers einkalkulieren

- [ ] **Folgeschritte planen**
  - [ ] Bei erfolgreicher Löschung: eigene Marke **WOVEN** in Klassen 9/42/45 anmelden
  - [ ] Bei Teilerfolg: Anmeldung mit enger Spezifikation („dating services“) + Koexistenzangebot
  - [ ] Bei Niederlage: Re-brand oder Lizenzverhandlungen mit Centinal, LLC



