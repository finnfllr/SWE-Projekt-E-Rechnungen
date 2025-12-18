# Lastenheft
## 1. Projektübersicht
Das System „Rechnung Digital“ ist eine Softwarelösung zur automatisierten Erstellung, Validierung und Versendung elektronischer Rechnungen (E-Rechnungen) gemäß der europäischen Norm EN 16931 (ZUGFeRD/XRechnung). Ziel ist es, den manuellen Aufwand in der Buchhaltung durch eine „No-Touch“-Automatisierung um bis zu 80 % zu reduzieren. Das System integriert sich als Middleware zwischen bestehenden ERP/CRM-Systemen und dem Versandkanal (E-Mail/Peppol) und überwacht den gesamten Zahlungszyklus bis hin zum Mahnwesen.

## 2. Funktionale Anforderungen
### 2.1. Use Cases
Das System unterscheidet zwischen vollautomatisierten Prozessen (Regelfall) und manuellen Eingriffen durch Mitarbeiter im Fehlerfall („Discovery Layer“).
![Use-Case-Diagramm](https://github.com/finnfllr/SWE-Projekt-E-Rechnungen/blob/main/Images/Use%20Case.png)

### 2.2. Prozessabläufe
Die Prozesslogik folgt einem „Management by Exception“-Ansatz. Der Standardprozess läuft ohne menschliche Interaktion ab.
![Swimlane-Diagramm](https://github.com/finnfllr/SWE-Projekt-E-Rechnungen/blob/main/Images/Swim%20Lanes.png)

## 3. Technische Anforderungen
### 3.1. Datenstruktur
Das System basiert auf einer serviceorientierten Architektur, die streng zwischen dem Domänen-Modell (Daten) und den Services (Logik) trennt.
![Klassendiagramm](https://github.com/finnfllr/SWE-Projekt/blob/main/Images/Klassendiagramm%20Allgemein.png)

Erläuterung der Komponenten:

- Domänen-Modell:

  - Zentrales Objekt ist die `Rechnung`, welche über eine eindeutige `rechnungsNr` (UUID) verfügt.

  - Der Lebenszyklus der Rechnung wird über das Enum `RechnungsStatus` gesteuert. Dieses bildet die logische Kette von der Erstellung (`ERSTELLT`) über die Prüfung (`VALIDIERUNG_OK` / `FEHLER`) bis hin zum Mahnwesen (`GEMAHNT_STUFE_1` bis `BEI_INKASSO`) ab .

  - Die Klasse `RechnungsFehler` speichert Validierungsprobleme, damit diese im Dashboard angezeigt werden können.

- Kern-Services (Core Engine):

  - Der `RechnungsManager` fungiert als Controller, der die Prozesse steuert.

  - Die Interfaces `IGenerator` und `IValidator` entkoppeln die konkrete Implementierung (ZUGFeRD, XML) von der Logik. Dies gewährleistet, dass später auch neue Standards (z. B. XRechnung 3.0) leicht integriert werden können.

  - Der `MahnService` prüft täglich die Fälligkeiten und eskaliert den Status der Rechnung bei Zahlungsverzug automatisch.

- Infrastruktur & Schnittstellen:

  - `ERPRepository`: Lädt die Rohdaten aus dem vorgeschalteten ERP-System.

  - `MailService`: Übernimmt den physischen Versand der generierten PDF/XML-Container via SMTP.

  -

### 3.2. Systemarchitektur
Das System wird als mehrschichtige Anwendung (Multi-Tier) realisiert :
1. Frontend (Dashboard): Webbasierte Oberfläche für Statusanzeige und Fehlerbehandlung.
2. Backend (Core Engine): Java-basierte Anwendung.
  -Nutzung von javax.xml.parsers für XML-Verarbeitung.
  -Validierung mittels javax.xml.validation gegen XRechnung-Schemata.
  -Währungsberechnungen präzise mittels java.math.BigDecimal.

3. Datenbank: Relationale Datenbank zur Speicherung von Rechnungen, Audit-Logs (Protokollierung) und Status.

4. Interfaces:
  -SMTP-Server (Jakarta Mail API) für den Versand.
  -REST/CSV-Schnittstellen zu ERP und Inkasso-Dienstleistern.

### 3.3. Sequenzdiagramme
Um die zeitliche Abfolge der Erstellung sicherzustellen, ist folgender Ablauf definiert:
![Seqzendiagramm-Erstellung](https://github.com/finnfllr/SWE-Projekt-E-Rechnungen/blob/main/Images/Sequenz%20Diagramm%20Rechnungserstellung.png)

Um die zeitliche Abfolge der Mahnung sicherzustellen, ist folgender Ablauf definiert:
![Sequenzdiagramm-Mahnung](https://github.com/finnfllr/SWE-Projekt-E-Rechnungen/blob/main/Images/SequenzDiagramm%20Mahnung.png)

## 4. Nicht-funktionale Anforderungen (Qualitätssicherung)
- Performance: Die Validierung einer Rechnung muss in unter 150 ms erfolgen, um auch Massenverarbeitungen nicht zu verzögern.
- Rechtssicherheit & Compliance: Das System muss 100 % konform zur Norm EN 16931 sein. Änderungen am Validierungsstandard (z. B. XRechnung 3.x) müssen durch Updates der Schemata einpflegbar sein.
- Zuverlässigkeit: Bei SMTP-Fehlern (z. B. Bounces, Server nicht erreichbar) muss eine automatische Wiederholungslogik (Retry 4xx) greifen.
- Audit-Trail: Jede Änderung am Status einer Rechnung sowie manuelle Eingriffe müssen unveränderbar protokolliert werden (Logging für GoBD).

## 5. Testplan
## Randinformationen mit denen wir arbeiten müssen
### Stand der Technik – Elektronische Rechnungssysteme
1. Überblick über den aktuellen Stand der Technik

Die elektronische Rechnung (E-Rechnung) hat sich in den letzten Jahren als Standard im B2B- und B2G-Umfeld etabliert. Moderne Systeme ermöglichen die Erstellung, Validierung, Übertragung und Archivierung von Rechnungen in strukturierten, maschinenlesbaren Formaten gemäß der europäischen Norm EN 16931.
In Deutschland sind insbesondere die Formate XRechnung und ZUGFeRD verbreitet.

Der Markt lässt sich in mehrere Kategorien einteilen: ERP-Systeme, Spezial-Rechnungstools, Netzwerk- und Plattformlösungen sowie Entwickler- und API-basierte Lösungen.
2. ERP- und Buchhaltungssysteme mit E-Rechnungsfunktion

ERP-Systeme integrieren die E-Rechnung direkt in betriebliche Kernprozesse (Order-to-Cash, Purchase-to-Pay).

Beispiele:

SAP S/4HANA – https://www.sap.com

Microsoft Dynamics 365 Business Central – https://www.microsoft.com/dynamics

DATEV Unternehmen online – https://www.datev.de

Lexware Office – https://www.lexware.de

sevDesk – https://sevdesk.de

Technischer Stand:

Automatische Rechnungserstellung aus Buchhaltungsdaten

Unterstützung von XRechnung und ZUGFeRD

Teilweise integrierte Validierung

Stark prozess- und benutzergetrieben

Einschränkung:
Hohe Kosten, komplexe Konfiguration, meist nicht vollständig automatisiert (manuelle Freigaben).

3. Spezialisierte E-Rechnungstools

Diese Tools fokussieren sich ausschließlich auf die Erstellung und den Versand von Rechnungen.

Beispiele:

easybill – https://www.easybill.de

Billomat – https://www.billomat.com

FastBill – https://www.fastbill.com

WISO MeinBüro – https://www.buhl.de/meinbuero

Technischer Stand:

Webbasierte Oberflächen

Formularbasierte Rechnungserstellung

Export als ZUGFeRD / XRechnung

Geringe Einstiegshürden

Einschränkung:
Stark UI-lastig, kaum geeignet für vollautomatische Prozesse.

4. Netzwerk- und Plattformlösungen (PEPPOL & EDI)
Diese Systeme übernehmen den strukturierten Rechnungsaustausch zwischen Unternehmen und Behörden.

Beispiele:

Basware – https://www.basware.com

Pagero – https://www.pagero.com

Sovos – https://www.sovos.com

OpenPEPPOL – https://peppol.org

Technischer Stand:

Netzwerkbasierter Austausch

Validierung vor Versand

Hohe Compliance (B2G)

Einschränkung:
Komplex, teuer, primär für Großunternehmen.

5. Entwickler- & API-basierte Lösungen

Diese Ansätze sind besonders relevant für moderne Softwareprojekte.

Beispiele:

Mustangproject (Java) – https://www.mustangproject.org

OpenXRechnung – https://github.com/itplr-kosit

B2Brouter – https://www.b2brouter.net

Crossinx – https://www.crossinx.com

Technischer Stand:

Programmgesteuerte Erstellung von XML-Rechnungen

Automatische Validierung

API-Integration in eigene Systeme

Vorteil:
Hohe Automatisierung, ideal für Zero-Touch-Workflows.

### Mock-ups: Ideen & Inspiration
Sinnvolle Mock-up-Screens (empfohlen):

Automatischer Rechnungsworkflow (Dashboard)

Validierungsstatus (grün / rot / Warnungen)

XML-Vorschau + PDF-Vorschau

Versandstatus (SMTP / PEPPOL)

Fehlerprotokoll & Logs

Systemkonfiguration (keine Benutzeraktion!)
### Inspiration aus bestehenden Tools (UI-Beispiele)

<img width="1208" height="808" alt="Bildschirmfoto 2025-12-18 um 20 02 04" src="https://github.com/user-attachments/assets/f64e6f12-d8b6-4db9-983a-a69dec5cf93a" />

<img width="1485" height="787" alt="Bildschirmfoto 2025-12-18 um 20 02 26" src="https://github.com/user-attachments/assets/c547f052-7f2c-4d2e-9eea-479a1357b697" />

<img width="637" height="380" alt="Bildschirmfoto 2025-12-18 um 20 03 37" src="https://github.com/user-attachments/assets/765f7f4b-0586-474b-a88f-692b109bafa6" />

<img width="1019" height="648" alt="Bildschirmfoto 2025-12-18 um 20 06 33" src="https://github.com/user-attachments/assets/94fb18e5-e750-4598-baeb-092b871e1323" />

<img width="1024" height="679" alt="Bildschirmfoto 2025-12-18 um 20 06 57" src="https://github.com/user-attachments/assets/134381ce-0b10-4cd8-ab53-7d2d783116f2" />

<img width="759" height="505" alt="Bildschirmfoto 2025-12-18 um 20 07 23" src="https://github.com/user-attachments/assets/1ae632af-a6ae-4a0c-b44e-3f974e798d67" />

<img width="1032" height="573" alt="Bildschirmfoto 2025-12-18 um 20 07 46" src="https://github.com/user-attachments/assets/a1ff5fb1-afc6-4e48-964f-8e2fa51ddca1" />

<img width="248" height="499" alt="Bildschirmfoto 2025-12-18 um 20 08 49" src="https://github.com/user-attachments/assets/c7a4300c-e39a-4553-a048-061f3778f0a1" />

<img width="1485" height="787" alt="Bildschirmfoto 2025-12-18 um 20 02 26" src="https://github.com/user-attachments/assets/9757a8bc-be56-431f-a691-9dd75c92f2d2" />
<img width="800" height="468" alt="Bildschirmfoto 2025-12-18 um 20 20 36" src="https://github.com/user-attachments/assets/aba76327-f37e-47af-b880-2db606e8a54b" />

<img width="854" height="476" alt="Bildschirmfoto 2025-12-18 um 20 22 14" src="https://github.com/user-attachments/assets/2e6cf779-a247-4358-9b00-a5e417cfdd33" />

<img width="870" height="481" alt="Bildschirmfoto 2025-12-18 um 20 22 31" src="https://github.com/user-attachments/assets/559c0e6a-574d-4548-9002-98a1135c01c2" />

<img width="884" height="486" alt="Bildschirmfoto 2025-12-18 um 20 23 43" src="https://github.com/user-attachments/assets/4526e631-a5e6-4cdd-afd2-f6686c311fc0" />
<img width="879" height="482" alt="Bildschirmfoto 2025-12-18 um 20 23 35" src="https://github.com/user-attachments/assets/bd9e4870-ccfa-4125-8a8b-c22300511ae7" />

<img width="877" height="471" alt="Bildschirmfoto 2025-12-18 um 20 24 31" src="https://github.com/user-attachments/assets/cee7d084-7955-40b4-9fcf-023c2c82003a" />

<img width="873" height="588" alt="Bildschirmfoto 2025-12-18 um 20 24 52" src="https://github.com/user-attachments/assets/1bdda49c-f17b-49e2-b872-dc22d033d88f" />

<img width="878" height="490" alt="Bildschirmfoto 2025-12-18 um 20 25 27" src="https://github.com/user-attachments/assets/ff41ca1e-41de-43d3-8537-ade0bd485cfd" />

<img width="896" height="483" alt="Bildschirmfoto 2025-12-18 um 20 25 48" src="https://github.com/user-attachments/assets/6acb1d77-7a47-4935-ad3b-655b198516a2" />















