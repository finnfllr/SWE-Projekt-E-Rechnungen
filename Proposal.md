# Proposal

# Rechnung 4.0
App für die Erstellung von E-Rechnungen

### Mitglieder
- Finn Feller, 1439832
- Noel Hidenbrand, 1570029
- Gian-Luca Sarcone, 1574340
- Mulugeta Gebregergis, 1574762
- Nikola Kostic, 1590731
- Nurettin Tasoluk, 1522479

---

# Inhaltsverzeichnis

1. Abstract: Rechnung 4.0
2. Produktbeschreibung: „Rechnung 4.0“  
3. Mindmap  
4. Gesetzliche Anforderungen  
5. Systemarchitektur  
5.1 Beispiel Szenario  
5.2.1 Mahnungs-Szenario  
5.2.2 Technische Vorgehensweise der Mahnung  
6. Technologien  
7. Business Model Canvas  
8. Gantt Diagramm  
9. Aufwandsabschätzung und benötigte Ressourcen  
10. Projektstruktur & Aufwände
11. Zentrale Eigenschaften  
12. Entwicklungsphasen 
13. Zielgruppe  
13.1 Erwartungshaltung der Zielgruppe  
14. Messlatte für das Produkt (Erfolgskriterien)  
15. Fazit/Konkurrenzanalyse

---

# 1. Abstract: Rechnung 4.0

Dieses Produkt umfasst ein Softwaresystem zur automatisierten Erzeugung elektronischer Rechnungen im ZUGFeRD-Format, wobei die benötigten Produktdaten aus dem ERP-System stammen. Ergänzend werden alle kundenbezogenen Informationen wie Stammdaten, VAT-Details oder Rabattstaffeln direkt aus dem angebundenen CRM-System eingebunden. Anschließend werden die Rechnungen selbstständig per E-Mail versendet, wobei der erfolgreiche Eingang auf dem Server des Empfängers verifiziert wird. Die Software nutzt etablierte Bibliotheken zur XML-Verarbeitung, PDF-Erstellung und Schema-Validierung, wodurch eine formale und inhaltliche Richtigkeitsprüfung gemäß dem Format EN 16931 sichergestellt wird. Nach Ablauf der Zahlungsfrist überwacht das System automatisiert den Zahlungseingang. Bleibt dieser aus, wird zunächst ein interner Hinweis zum ausstehenden Betrag erzeugt und anschließend eine gestaffelte Mahnfolge ausgelöst. Erfolgt auch nach der dritten Mahnung keine Zahlung, wird der Vorgang automatisch an die Rechtsabteilung zur weiteren Bearbeitung übergeben.

---
# 2. Produktbeschreibung: „Rechnung 4.0“

![Produktbeschreibung](https://github.com/finnfllr/SWE-Projekt-E-Rechnungen/blob/main/Images/Produktbeschreibung.png)

1. **Allgemeine Beschreibung**
Das geplante Produkt trägt den Titel „Automatisches E-Rechnungsmodul“ und umfasst ein Softwaresystem zur Erstellung und Verarbeitung elektronischer Rechnungen. Die Anwendung erfüllt die in Deutschland geltenden Anforderungen an die elektronische Rechnungsstellung und orientiert sich an den etablierten Standards XRechnung und ZUGFeRD. Damit werden die Vorgaben der EN 16931 sowie die Richtlinien der EU-Richtlinie 2014/55/EU eingehalten.
Als Grundlage nutzt das System vorhandene Unternehmensdaten, beispielsweise aus einem ERP-System. Auf dieser Basis entsteht eine verlässliche Lösung zur normgerechten Erzeugung elektronischer Rechnungen.

2. **Funktionaler Schwerpunkt**

Der funktionale Kern des Systems liegt in der durchgehenden Automatisierung des Rechnungsprozesses. Dazu gehören:

## 2.1 Erstellung der Rechnung**

- Übernahme der relevanten Daten aus dem ERP-System

- Generierung des strukturierten Rechnungsformats gemäß EN 16931

- Aufbereitung der Informationen in maschinenlesbarer Form (XML)

## 2.2 Validierung**

- Prüfung der formalen Anforderungen des Standards

- Kontrolle der Pflichtfelder sowie der strukturellen Vorgaben

- Überprüfung der fachlichen Regeln (z. B. Steuersätze, Beträge, Datumsangaben)

## 2.3 Versand und Zustellungsprüfung**

- automatischer Versand der Rechnung per E-Mail

- Auswertung der Rückmeldungen des Mailservers

- Zuordnung des technischen Zustellstatus zur jeweiligen Rechnung



---

# 3. Mindmap

![Mindmap](https://github.com/finnfllr/SWE-Projekt/blob/main/Images/Mindmap.png)
*Abbildung: Mindmap zum E-Rechnungsprojekt*


---

# 4. Gesetzliche Anforderungen

Für die Entwicklung der E-Rechnungslösung gelten die Vorgaben des Umsatzsteuergesetzes und der europäischen Norm EN 16931.  
Nach § 14 UStG müssen u. a. folgende Angaben enthalten sein:

- Vollständiger Name und Anschrift von Absender und Empfänger
- Steuernummer oder Umsatzsteuer-Identifikationsnummer
- Menge und Art der gelieferten Gegenstände
- Zeitpunkt der Lieferung oder Leistung
- Rechnungsnummer und Ausstellungsdatum
- Netto- und Bruttobetrag
- Steuerbetrag und Steuersatz
- Entgeltminderungen

**Anforderungen an E-Rechnungen gemäß EN 16931:**

***Strukturierte maschinenlesbare Daten***  
  
  - Rechnung muss in einem strukturierten elektronischen Format vorliegen (z. B. ZUGFeRD)

***Interoperabilität***  
 
- Verwendung standardisierter Codelists (z. B. für Währungen, Umsatzsteuerkategorien)

- Strikte Einhaltung des Datenmodells der EN 16931

***Semantische Validierung***  

- Die erzeugte Rechnung muss den Validatoren der öffentlichen Verwaltung entsprechen (z. B. KoSIT-Validator für XRechnung).

Die Einhaltung dieser gesetzlichen Vorgaben ist erforderlich, um die Rechtssicherheit der Rechnung zu gewährleisten und eine reibungslose, automatisierte Weiterverarbeitung zu ermöglichen.

---

# 5. Systemarchitektur

Die Architektur ist darauf ausgelegt, als „Discovery Layer“ zu fungieren, der Prozesse automatisiert und nur bei Fehlern eine menschliche Interaktion erfordert.

**1. Frontend (Dashboard):** Benutzeroberfläche zur Statusanzeige, Fehlerkorrektur und zum Monitoring der Mahnstufen.

**2. Backend (Core Engine):**

- **Datenübernahme:** Import aus ERP/CRM.  
- **XML-Generator:** Erstellung der Strukturdaten gemäß EN 16931.  
- **Validierungs-Engine:** Prüfung von Syntax, Schema und Business Rules (z. B. Steuersätze, Summen).

**3. Datenbank:**Revisionssichere Archivierung, Logging und Speicherung des Zahlungsstatus.


**4. Schnittstellen (APIs) & Services:**

- SMTP-Server für den E-Mail-Versand.  
- Anbindung an ERP und CRM Systeme.

---

## 5.1 Beispiel Szenario

1. **Datenverfügbarkeit:** Im ERP-System liegen die notwendigen Daten vor; das System erkennt automatisch den Handlungsbedarf.  
2. **Generierung:** Das System erzeugt im Hintergrund eine XML-Rechnung (ZUGFeRD/XRechnung) ohne manuellen Eingriff.  
3. **Validierung:** Das Modul prüft Pflichtfelder (z. B. BT-1, BT-5) und logische Regeln.  
   - **Fehlerfall:** Der Vorgang erscheint im Dashboard zur Korrektur durch den Mitarbeiter.  
   - **Erfolgsfall:** Der Prozess läuft weiter.  
4. **Versand:** Die Rechnung wird automatisch per E-Mail an den Kunden gesendet.  
5. **Zahlungskontrolle:** Nach Fristablauf überprüft das System den Zahlungseingang und leitet bei ausbleibender Zahlung den Mahnprozess ein.

---

## 5.2.1 Mahnungs-Szenario

1. **Überprüfung:** Das System führt nächtlich eine Prüfung auf offene Posten durch.  
2. **Übergabe:** Ein Mitarbeiter bekommt alle offenen Mahnungen und prüft, ob diese an die Kunden übermittelt werden sollen.  
3. **Einstufung:** Die Mahnungen werden nach dem Absenden neu eingestuft, sodass der Status und die nächste Aktion (bspw. Inkasso) klar ist.

---

## 5.2.2 Technische Vorgehensweise der Mahnung

1. Status: Überfällige Rechnungen wechseln in „ZUR_MAHNUNG_VORLEGEN“.  
2. Die erste Erinnerung führt zu „GEMAHNT_STUFE_1“.  
3. Erfolgt 14 Tage lang keine Zahlung, erfolgt „GEMAHNT_STUFE_2“.  
4. Bei Klärungsbedarf kann der Vorgang auf „ZURÜCKGESTELLT“ gesetzt werden; nach Abschluss der Klärung geht er zurück in Stufe 1.  
5. Nach einer weiteren Frist erfolgt die Einstufung in „GEMAHNT_STUFE_3“.  
6. In dieser Stufe kann der Vorgang automatisch an die Rechtsabteilung übergeben werden.  
7. Nach Zahlungseingang wird der Vorgang unabhängig von der Stufe auf „BEZAHLT“ gesetzt.

---

# 6. Technologien

**1. Programmiersprache & Core-APIs**  
- Sprache: Java.  
- Basis-APIs (JDK):  
  - `java.math.BigDecimal`: API für präzise Währungsberechnungen.  
  - `java.time.LocalDate`: API für Datums- und Fristenmanagement.  
  - `java.util.regex`: API für Formatvalidierungen (RegEx).

**2. XML-Verarbeitung & Validierungs-APIs**  
- Parsing-API: `javax.xml.parsers` (Verarbeitung von XML-Strukturen).  
- Validierungs-API: `javax.xml.validation` (Prüfung gegen XRechnung-Schemata).  
- SAX-API: `org.xml.sax` (Ereignisbasierte XML-Verarbeitung für Performance).

**3. Kommunikation & Messaging APIs**  
- Mail-API: Jakarta Mail API (Standard für den E-Mail-Versand).  
- MIME-Handling: `jakarta.activation` (API zur Verarbeitung von Dateianhängen/MIME-Typen).  
- Protokoll: SMTP (Simple Mail Transfer Protocol) für den Transport.

**4. Datenstandards & Formate**  
- Rechnungsstandards: ZUGFeRD, XRechnung, EN 16931.  
- Dateiformate:  
  - XML: Primärformat für Rechnungsdaten und Inkasso-Export.  
  - PDF: Sichtformat für den Kunden.  
  - CSV: Exportformat für externe Dienstleister (Inkasso).

**5. Infrastruktur-Komponenten**  
- Datenbank: Speicherung von Rechnungen und Status (impliziert durch SQL-Logik `getAlleOffenenRechnungen`).  
- Scheduler/Timer: API oder Dienst für den täglichen nächtlichen Job (Tagesabschluss).

---

# 7. Business Model Canvas

![The Business Model Canvas](https://github.com/finnfllr/SWE-Projekt/blob/main/Images/BusinessCanvas.jpg)
*Abbildung: The Business Model Canvas*

---

# 8. Gantt Diagramm

![Gantt-Diagramm](https://github.com/finnfllr/SWE-Projekt/blob/main/Images/Ganttdiagramm.jpg)
*Abbildung: Gantt-Diagramm des Projektplans*

---

# 9. Aufwandsabschätzung und benötigte Ressourcen (Zeit, Material, Mann-Tage)

Für jede Phase wurden der zeitliche Bedarf, die erforderlichen Ressourcen und die benötigten Mann-Tage (1 Mann-Tag = 8 Stunden Arbeitstag) für ein Entwicklerteam aus 6 Personen bestimmt.

**Phasen-Übersicht (Zusammenfassung):**

- **Anforderungsanalyse und Planung**  
  - Aufwand: 2 Wochen  
  - 60 Mann-Tage  
  - Ressourcen: Dokumentationstools, gemeinsame Workshops

- **Design**  
  - Aufwand: 3 Wochen  
  - 90 Mann-Tage  
  - Ressourcen: UML-Diagrammtools, Architektur-Frameworks

- **Implementierung**  
  - Aufwand: 6 Wochen  
  - 180 Mann-Tage  
  - Ressourcen: Entwicklungstools, Git für Versionskontrolle, Softwarelizenzen (ZUGFeRD Parsing), Code Repositories

- **Test und Qualitätssicherung**  
  - Aufwand: 3 Wochen  
  - 90 Mann-Tage  
  - Ressourcen: Test Tools, Mock-Umgebungen

- **Deployment und Dokumentation**  
  - Aufwand: 1 Woche  
  - 30 Mann-Tage  
  - Ressourcen: Deploymentskripte, Serverzugänge

Zusammenfassend lässt sich sagen, dass die Projektdauer auf 15 Wochen beläuft und das Team 450 Mann-Tage aufwendet. Neben den spezifischen Ressourcen werden grundlegende Materialien benötigt wie Standard Hardware, eine stabile Internetverbindung, Kommunikationstools und Meetingräume für Besprechungen. Die ergänzenden Ressourcen schaffen die technischen und organisatorischen Grundlagen, um eine effiziente Zusammenarbeit zu gewährleisten.  

---

# 10. Projektstruktur & Aufwände

### **Vorgehensmodell & Iterationen**

Die Entwicklung erfolgt iterativ in kleinen Abschnitten, um Flexibilität zu gewährleisten.

- **Interne Iterationen:** Regelmäßige Abstimmung im Team zu Modellierung, Tests und Umsetzung.  
- **Externe Iterationen:** Feedbackschleifen mit „simulierten Kunden“ zur Prüfung der Anforderungen und Diagramme.  
- **Entwicklungsnahe Iterationen:** Übersetzung von Anforderungen in User-Stories (z. B. „Als Sachbearbeiter möchte ich...“).

---

# 11. Zentrale Eigenschaften

1. **Vollautomatisierte Erstellung:** Generierung des strukturierten Rechnungsformats (XML) und Übernahme relevanter Daten aus dem ERP.  
2. **Integrierte Validierung:** Prüfung auf formale (Schema) und inhaltliche Korrektheit (Business Rules) vor dem Versand.  
3. **Zustellungsprüfung & Versand:** Automatischer E-Mail-Versand inklusive Auswertung technischer Rückmeldungen.  
4. **Zahlungsüberwachung & Mahnwesen:** Automatische Fristenkontrolle und Eskalation in definierte Mahnstufen (Erinnerung → Stufe 1 → Stufe 2 → Rechtsabteilung).  
5. **Dashboard & Monitoring:** Übersichtliche Darstellung des Status und Meldung von Fehlern, die manuelle Korrekturen erfordern.

---

# 12. Entwicklungsphasen

### **Release 1:** Core Engine & Validierung (MVP)  
**Fokus:** Sicherstellung der Normkonformität und korrekten Dateierzeugung.  

- **Daten-Mapping:** Implementierung der Schnittstelle zur Übernahme von Rohdaten aus dem ERP-System.  
- **XML-Generierung:** Entwicklung des Parsers zur Erstellung valider ZUGFeRD- und XRechnung-Dateien mittels `javax.xml.parsers`.  
- **Validierungs-Logik:**  
  - Einbau der Schema-Validierung (`javax.xml.validation`) gegen offizielle XSDs.  
  - Implementierung grundlegender Business-Rules (z. B. Steuerberechnung mit `java.math.BigDecimal`).  

- **Output:** Das System kann eine rechtssichere E-Rechnung (XML + PDF) erzeugen und lokal speichern.

### **Release 2:** Automatisierung & Versand ("No-Touch")  
**Fokus:** Integration in den Workflow und Kommunikation.

- **E-Mail-Integration:** Implementierung des SMTP-Versands mittels Jakarta Mail API.  
- **Status-Handling:** Verarbeitung von Server-Rückmeldungen (Bounces, Status 250 OK) und Zuordnung zur Rechnung.  
- **Dashboard (Frontend):**  
  - Erstellung der Benutzeroberfläche für das Monitoring.  
  - Anzeige von Validierungsfehlern ("Discovery Layer"), damit Mitarbeiter eingreifen können, falls Daten fehlen.  
- **Archivierung:** Automatisierte, revisionssichere Ablage der versendeten Dateien.

### **Release 3:** Mahnwesen & Lifecycle Management  
**Fokus:** Was passiert nach dem Versand? (Zahlungsüberwachung).

- **Timer-Service (Nightly Job):** Implementierung des täglichen Jobs ("starteTagesabschluss"), der alle offenen Rechnungen prüft.  
- **Fristen-Logik:** Berechnung der Verzugstage und automatische Statussetzung:  
  - Erinnerung ("Haben Sie uns vergessen?").  
  - Mahnstufe 1 & 2 (mit Gebührenberechnung).  
- **Inkasso-Schnittstelle:** Entwicklung des CSV/XML-Exports für den Inkasso-Dienstleister bei erfolglosem Mahnlauf (Stufe 3).  
- **Manuelle Eingriffsmöglichkeiten:** Funktionen im Dashboard zum "Zurückstellen" oder "Bestätigen" von Mahnvorschlägen.

---

# 13. Zielgruppe

**Primär:** Kleine und mittlere Unternehmen (KMU), Steuerberater, Buchhaltungsbüros und öffentliche Auftragnehmer im B2B-Sektor.

**Sekundär:** Softwarehersteller (als Modul-Integration) und E-Commerce-Unternehmen.

## 13.1 Erwartungshaltung der Zielgruppe

- **Rechtssicherheit:** Einhaltung der gesetzlichen Anforderungen (§ 14 UStG, EN 16931).  
- **Automatisierung:** Reduktion manueller Eingaben und Fehlerquellen.  
- **Integration:** Einfache Anbindung an vorhandene ERP-Systeme.  
- **Transparenz:** Lückenlose Nachvollziehbarkeit und revisionssichere Archivierung.

---

# 14. Messlatte für das Produkt (Erfolgskriterien)

- **Performance:** Validierungszeiten unter 150 ms.  
- **Stabilität:** Unveränderbares Logging und Wiederholungsversuche bei SMTP-Fehlern.  
- **Effizienz:** Reduktion von Kosten, Fehlern und manuellen Arbeitsschritten um bis zu 80 %.  
- **Konformität:** 100 % normkonforme E-Rechnungen gemäß EN 16931.

---

# 15. Fazit / Konkurrenzanalyse

Auf dem Markt existieren diverse Produkte, die die Erstellung von E-Rechnungen ermöglichen. Diese Produkte konzentrieren sich jedoch in der Regel auf spezifische Aspekte, wie beispielsweise die Erstellung und den Versand, vernachlässigen jedoch die Überwachung von Fristen und die Validierung. Unser Produkt bietet eine umfassende Lösung, die all diese Aspekte abdeckt, und zeichnet sich durch eine benutzerfreundliche Bedienung aus, da sämtliche Prozesse vollautomatisiert ablaufen. Die Mitarbeiter werden über fällige Zahlungen informiert und haben die Möglichkeit, E-Rechnungen manuell über eine Web-Anwendung zu erstellen.

---




