# Lastenheft
## 1. Projektübersicht
Das System „Rechnung Digital“ ist eine Softwarelösung zur automatisierten Erstellung, Validierung und Versendung elektronischer Rechnungen (E-Rechnungen) gemäß der europäischen Norm EN 16931 (ZUGFeRD/XRechnung). Ziel ist es, den manuellen Aufwand in der Buchhaltung durch eine „No-Touch“-Automatisierung um bis zu 80 % zu reduzieren. Das System integriert sich als Middleware zwischen bestehenden ERP/CRM-Systemen und dem Versandkanal (E-Mail/Peppol) und überwacht den gesamten Zahlungszyklus bis hin zum Mahnwesen.

## 2. Funktionale Anforderungen
### 2.1. Use Cases
| Attribut | Beschreibung |
| :--- | :--- |
| **Autor** | Finn Feller, Mulugeta Gebregergis, Nikola Kostic, Nurettin Tasoluk, Noel Hildenbrand, Gian-Luca Sarcone |
| **Kurzbeschreibung** | Rechnungserstellung |
| **Primärer Akteur** | ERP-System, Zeitsteuerung |
| **Sekundärer Akteur** | Mitarbeiter |
| **Auslöser, Vorbedingung** | Zeit |
| **Ergebnisse, Nachbedingungen** | E-Rechnung in ZUGFeRD |
| **Technische Randbedingungen** | 1. ERP-System<br>2. Trigger<br>3. Sytem muss laufen |
| **Allgemeine Bemerkungen** | funktionsfähig |
| **Offene Punkte** | |

### Ablauf

| Schritt | Akteur | Ablauf | Verzweigung |
| :--- | :--- | :--- | :--- |
| **1** | **Anmeldung** | | |
| 1.1 | ERP | Rechnung erstelllen | Wenn Validierung fehlschlägt → A1 |
| 1.2 | ERP | Validierund durschüfhren |  |
| 1.3 | ERP | E-Rechnung versenden |  |
| 1.4 | Zeitsteuerung | Zahlungeingang prüfen | Wenn Zahlung ausbleibt → E1 |
| **Ausnahmen** |  |  |  |
| A1 | Mitarbeiter | Validierungsfehler korriegieren |  |
| A2 | Mitarbeiter | Mahnung freigeben |  |
| **Erweiterung** |  |  |  |
| E1 | Zeitsteureung | Initialisierung der Mahnung (inkl. Mahnprozess anstoßen) |  |


Das System unterscheidet zwischen vollautomatisierten Prozessen (Regelfall) und manuellen Eingriffen durch Mitarbeiter im Fehlerfall.
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
  ![Zustandsautomat](https://github.com/finnfllr/SWE-Projekt-E-Rechnungen/blob/main/Images/Zustandsautomat.png)


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
- Präzision: Alle Währungsberechnungen müssen mittels java.math.BigDecimal durchgeführt werden, um Rundungsfehler auszuschließen
- Skalierbarkeit: Die Software muss leicht erweiterbar sein, um steigende Rechnungsvolumina, sowie Einbidnung neuer Mandanten oder Funktionen ohne Performance-Verlust zu unterstützen. 
- Rechtssicherheit & Compliance: Das System muss 100 % konform zur Norm EN 16931 sein. Änderungen am Validierungsstandard (z. B. XRechnung 3.x) müssen durch Updates der Schemata einpflegbar sein.
- Zuverlässigkeit: Bei SMTP-Fehlern (z. B. Bounces, Server nicht erreichbar) muss eine automatische Wiederholungslogik (Retry 4xx) greifen.
- Audit-Trail: Jede Änderung am Status einer Rechnung sowie manuelle Eingriffe müssen unveränderbar protokolliert werden (Logging für GoBD).
- Gesetzliche Wartbarkeit: Das System muss modular geschaffen sein, dass Gesetztesänderungen durch Updates der Validierungsschemata  ohne Änderung der Kernlogik erweitert werden können. 

## 4. Nicht-funktionale Anforderungen (Qualitätssicherung)

Dieses Kapitel beschreibt die nicht-funktionalen Anforderungen an das System zur automatisierten Erzeugung elektronischer Rechnungen. Nicht-funktionale Anforderungen definieren qualitative Eigenschaften des Systems und stellen sicher, dass die funktionalen Anforderungen unter realen Einsatzbedingungen zuverlässig, korrekt und regelkonform erfüllt werden. Sie sind maßgeblich für die Qualität, Wartbarkeit und Abnahmefähigkeit des Gesamtsystems.

## 4.1 Performance

Die Performance des Systems ist so auszulegen, dass auch bei hohem Rechnungsaufkommen eine reibungslose und verzögerungsfreie Verarbeitung gewährleistet ist. Insbesondere die Validierung einer Rechnung muss innerhalb von unter 150 Millisekunden erfolgen. Diese Anforderung ist notwendig, um Massenverarbeitungen, beispielsweise im Rahmen von Tages- oder Monatsläufen, ohne spürbare Verzögerungen durchführen zu können.

Eine konstante und vorhersehbare Validierungsdauer stellt sicher, dass das System sowohl im Einzelbetrieb als auch bei der Verarbeitung großer Rechnungsvolumina effizient arbeitet. Performance-Einbußen bei steigender Last sind zu vermeiden, da sie den automatisierten Rechnungsprozess erheblich beeinträchtigen würden.

## 4.2 Präzision

Die korrekte Verarbeitung von Geldbeträgen stellt eine zentrale Qualitätsanforderung dar. Alle Währungs- und Betragsberechnungen müssen daher konsequent unter Verwendung von java.math.BigDecimal durchgeführt werden. Dadurch wird sichergestellt, dass Rundungsfehler, wie sie bei der Verwendung von Gleitkommazahlen auftreten können, ausgeschlossen werden.

Diese Anforderung ist insbesondere im Kontext steuerlich relevanter Rechnungsdaten von hoher Bedeutung, da bereits geringe Abweichungen zu fehlerhaften Rechnungen oder Inkonsistenzen führen können. Die präzise Berechnung aller Beträge trägt somit maßgeblich zur inhaltlichen Korrektheit und rechtlichen Verlässlichkeit der erzeugten Rechnungen bei.

## 4.3 Skalierbarkeit

Das System muss so konzipiert sein, dass es leicht erweiterbar ist und mit wachsenden Anforderungen Schritt halten kann. Dazu gehört insbesondere die Fähigkeit, steigende Rechnungsvolumina zu verarbeiten, zusätzliche Kunden einzubinden sowie neue funktionale Erweiterungen zu integrieren, ohne dass es zu Performance verlusten kommt.

Die Skalierbarkeit stellt sicher, dass das System nicht nur für den initialen Einsatz geeignet ist, sondern auch langfristig genutzt und weiterentwickelt werden kann. Erweiterungen dürfen keine negativen Auswirkungen auf bestehende Funktionen oder die Systemstabilität haben. Ziel ist eine flexible Architektur, die Wachstum unterstützt, ohne grundlegende Anpassungen am Gesamtsystem erforderlich zu machen.

## 4.4 Rechtssicherheit und Compliance

Das System muss vollständig und zu 100 % konform zur Norm EN 16931 sein. Die Einhaltung dieser Norm ist Voraussetzung für eine rechtssichere elektronische Rechnungsstellung und stellt sicher, dass die erzeugten Rechnungen den geltenden gesetzlichen und fachlichen Anforderungen entsprechen.

Änderungen an den zugrunde liegenden Validierungsstandards, beispielsweise neue Versionen bestehender Rechnungsschemata, müssen durch die Aktualisierung der entsprechenden Schemadateien integrierbar sein. Dabei darf keine Anpassung der Kernlogik des Systems erforderlich sein. Diese Anforderung gewährleistet, dass das System auch bei zukünftigen Anpassungen der Standards weiterhin regelkonform eingesetzt werden kann.

## 4.5 Zuverlässigkeit

Ein hoher Grad an Zuverlässigkeit ist für den automatisierten Rechnungsprozess essenziell. Insbesondere beim Versand elektronischer Rechnungen muss das System in der Lage sein, mit temporären technischen Problemen umzugehen. Treten beim Versand SMTP-Fehler auf, beispielsweise durch nicht erreichbare Server oder sogenannte Bounces, muss automatisch eine Wiederholungslogik greifen.

Bei temporären Fehlern der Kategorie 4xx ist ein erneuter Versandversuch auszulösen, ohne dass ein manueller Eingriff erforderlich ist. Dadurch wird sichergestellt, dass Rechnungen nicht verloren gehen und der Versandprozess auch bei kurzzeitigen Störungen stabil und zuverlässig bleibt.

## 4.6 Audit-Trail

Zur Sicherstellung der Nachvollziehbarkeit und zur Erfüllung gesetzlicher Dokumentationsanforderungen muss das System einen vollständigen und unveränderbaren Audit-Trail bereitstellen. Jede Änderung am Status einer Rechnung, von der Erstellung über die Validierung bis hin zu Mahn- oder Eskalationsschritten, ist lückenlos zu protokollieren.

Darüber hinaus müssen auch manuelle Eingriffe, beispielsweise bei der Fehlerbehandlung oder bei Entscheidungen im Mahnwesen, nachvollziehbar erfasst werden. Die Protokollierung dient der internen Kontrolle sowie der externen Prüfung und darf nachträglich nicht manipuliert werden.

## 4.7 Gesetzliche Wartbarkeit

Das System muss modular aufgebaut sein, sodass gesetzliche oder normative Änderungen ohne Eingriffe in die bestehende Kernlogik umgesetzt werden können. Insbesondere Anpassungen an gesetzlichen Vorgaben oder Validierungsregeln sollen ausschließlich durch Updates der entsprechenden Validierungsschemata erfolgen.

Diese gesetzliche Wartbarkeit stellt sicher, dass das System langfristig einsetzbar bleibt und auf neue Anforderungen reagieren kann, ohne dass umfangreiche technische Umbauten notwendig werden. Dadurch werden Wartungsaufwand und Fehleranfälligkeit reduziert und die Zukunftssicherheit des Systems erhöht.

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


## Produkt und Projektdokumentation
<img width="1024" height="322" alt="Aufgabe 4_Erstellung der Produkt und Projektdokumentation_deliverry" src="https://github.com/user-attachments/assets/10455da0-2403-4770-b304-630bff2011fb" />












