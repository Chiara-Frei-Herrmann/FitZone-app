# SPEC.md · FitZone – Online-Kursbuchung

**Kundin:** Lisa Sommer, Inhaberin von FitZone (Gründung 2023)  
**Quelle:** 20 Fragen und Antworten im Kunden-Chat (Fragen 1–12 = Meeting #1, Fragen 13–20 = Meeting #2, nach Aufgabenaufteilung).  
**Stand:** 08.11.2026 

## Problem und Ziel

FitZone hat etwa 600 aktive Mitgliedschaften, sechs Trainer und 25–30 Kurstermine pro Woche. Kurse und Teilnehmer werden bisher in Excel geführt; Buchungen, Stornierungen und Trainerabsprachen erfolgen über WhatsApp. Die Warteliste ist nicht zuverlässig dokumentiert, Absagen werden verspätet weitergegeben und die Kursplanung kostet Lisa viel Zeit (Fragen 1, 2, 6, 7, 14).

**Ziel für Version 1:** mobil nutzbare Kursbuchung mit Tarifprüfung, automatisierter Warteliste, Kurs- und Trainerverwaltung sowie E-Mail-/Push-Benachrichtigungen bei wichtigen Ereignissen (Fragen 2, 12, 13, 20).

## Entitäten


| Entität | Attribute (Schlüssel fett) | Beleg |
|---|---|---|
| **Mitglied** | **mitglied_id**, name, email, mitgliedschaft_status, mitglied_seit | Fragen 10, 11, 16 |
| **Tarif** | **tarif_id**, bezeichnung (Basic/Plus/Premium), monatslimit, videozugang | Fragen 3, 15 |
| **Kursart** | **kursart_id**, bezeichnung, beschreibung, standarddauer | Fragen 6, 13 |
| **Kurstermin** | **termin_id**, beginn, ende, kapazitaet, status (geplant/abgesagt) | Fragen 6, 7 |
| **Trainer** | **trainer_id**, name, kontakt, beschaeftigungsart, qualifikationen | Fragen 6, 14, 16 |
| **Buchung** | **buchung_id**, gebucht_am, status, storniert_am, anwesenheit | Fragen 1, 4, 6, 11 |
| **Wartelisteneintrag** | **warteliste_id**, eingetragen_am, position, angebot_bis, status | Fragen 8, 9 |
| **Trainerverfügbarkeit** | **verfuegbarkeit_id**, start, ende, typ (verfügbar/abwesend), bemerkung | Frage 14 |


## Beziehungen

1. **Tarif 1:n Mitglied** – Ein Tarif gilt für viele Mitglieder; ein Mitglied hat aktuell einen Tarif (Fragen 3, 10).
2. **Kursart 1:n Kurstermin** – Ein Kurstyp kann an mehreren Terminen stattfinden; ein Termin gehört zu einer Kursart (Fragen 6, 7).
3. **Trainer 1:n Kurstermin** – Ein Trainer leitet mehrere Termine; jedem Termin wird im Grundmodell ein Trainer zugeordnet (Fragen 7, 14; zusätzliche Trainer bleiben Erweiterungsfall).
4. **Mitglied n:m Kurstermin über Buchung** – Ein Mitglied kann mehrere Kurstermine buchen; ein Termin kann mehrere Mitglieder haben. *Buchung* ist die verbindende Entität mit Datum und Status (Fragen 1, 2, 10).
5. **Mitglied 1:n Buchung** – Ein Mitglied besitzt mehrere Buchungsvorgänge (Fragen 1, 10).
6. **Kurstermin 1:n Buchung** – Ein Kurstermin kann mehrere Buchungen enthalten (Fragen 2, 6).
7. **Mitglied 1:n Wartelisteneintrag** – Ein Mitglied kann bei verschiedenen Terminen auf Wartelisten stehen (Fragen 8, 9).
8. **Kurstermin 1:n Wartelisteneintrag** – Pro vollem Kurstermin können bis zu fünf Wartelisteneinträge bestehen (Frage 8).
9. **Trainer 1:n Trainerverfügbarkeit** – Jeder Trainer trägt mehrere Zeitfenster oder Abwesenheiten ein (Frage 14).


## Business Rules

- **BR1 – Tarif und Buchungsberechtigung:** Nur Mitglieder mit aktiver und verifizierter Mitgliedschaft dürfen buchen. Basic erhält **maximal fünf Kursbuchungen pro Kalendermonat**, Plus und Premium unbegrenzt. Pausierte oder abgelaufene Mitgliedschaften dürfen keine neuen Kurse buchen (Fragen 3, 10, 11).
- **BR2 – Buchungszeitfenster:** Reguläre Kursbuchungen sind frühestens **14 Tage** und spätestens **eine Stunde vor Kursbeginn** möglich (Frage 11).
- **BR3 – Warteliste:** Bei vollem Kurs maximal **fünf Wartende** nach Eingangsreihenfolge. Wird mehr als zwei Stunden vor Beginn ein Platz frei, erhält die erste Person **30 Minuten** zur Bestätigung; andernfalls folgt die nächste. Innerhalb der letzten zwei Stunden erfolgt automatische Zuweisung mit Nachricht und **ohne No-Show- oder Stornogebührenfolge bei Nichterscheinen** (Fragen 8, 9).
- **BR4 – Stornierung und Kontingent:** Bis **zwei Stunden vor Kursbeginn** ist eine Stornierung kostenfrei; Basic erhält den Buchungsslot zurück. Bei späterer Stornierung bleibt der Basic-Slot verbraucht; für Basic/Plus ist eine Gebühr in Höhe von **50 % eines noch festzulegenden anteiligen Kurswerts** vorgesehen, Premium ist von Gruppenkurs-Stornogebühren ausgenommen. Sagt das Studio ab, wird der Basic-Slot wieder freigegeben (Fragen 4, 5, 7).
- **BR5 – No-Show-Sperre:** Nach **drei aufeinanderfolgenden unentschuldigten Nichterscheinen ohne Absage** wird die Live-Kursbuchung automatisch für **14 Tage** gesperrt. Trainer bestätigen dafür die Anwesenheit; Lisa kann Sperren bei begründetem Anlass aufheben. Kurzfristig automatisch zugewiesene Wartelistenplätze ohne Bestätigung zählen **nicht** als No-Show (Fragen 4, 6, 9, 19).

## Widersprüche

- **W1 – „Keine kostenpflichtigen Zusatzkurse“ vs. „50 % Kursgebühr bei später Stornierung“.** In Frage 3 erklärt Lisa, es gebe keine kostenpflichtigen Gruppenkurse. In Frage 4 nennt sie dennoch eine Stornogebühr auf Basis der Kursgebühr. **Auflösung (Frage 5):** Gruppenkurse sind im Mitgliedsbeitrag enthalten; die Stornogebühr soll aus einem **intern ermittelten anteiligen Kurswert** berechnet werden. Dieser Wert ist noch **nicht festgelegt**. Bis zur Klärung darf die App keine selbst erfundenen Beträge abbuchen.
- **W2 – „Online ist für alle Mitglieder zugänglich“ vs. „Basic hat keinen Zugang zur Videobibliothek“.** Lisa sagt in Frage 15, online sei für alle Mitglieder zugänglich, begrenzt gleichzeitig aber Videos auf Plus/Premium und hinterfragt in Frage 20 die Kommunikation des Basic-Zugangs. **Auflösung (Interpretation der Aussagen aus Fragen 2, 3, 15, 20):** **Zugang zur Kursbuchungs-App** erhalten alle aktiven Mitglieder, einschließlich Basic; **Zugang zur Videobibliothek** ist separat tarifabhängig. Ob einzelne Videos künftig für Basic freigegeben werden, muss Lisa später bestätigen.

## Prioritäten

| Priorität | Umfang | Quelle |
|---|---|---|
| **Version 1 – Muss** | Mobiler Kursplan und Selbstbuchung; tarifabhängige Berechtigung; Warteliste; Trainer-/Kursterminzuordnung; Benachrichtigungen über E-Mail und Push; einfache Mitgliederverwaltung | Fragen 2, 10, 12, 13, 20 |
| **Version 1 – unterstützend** | Trainer-Anwesenheitslisten, einfache Kursauslastungsübersicht, Verfügbarkeiten, Storno-/Sperrstatus; Import der vorhandenen Mitglieds- und Tarifdaten | Fragen 6, 14, 16, 19 |
| **Version 2** | Personal-Coaching-Buchung mit monatlich einem inkludierten Premium-Termin; Onlinezahlung weiterer Termine; Videobibliothek mit tarifabhängiger Freigabe | Fragen 15–18, 20 |
| **Später** | Erweiterte Auslastungsempfehlungen und Geburtstagsfunktionen | Fragen 14, 19, 20 |


## Offene Fragen

- **Kursstornogebühr:** Konkreter anteiliger Kurswert, rechtliche und buchhalterische Prüfung sowie Einzug sind offen (Frage 5).
- **No-Show-Details:** Ob die Sperre für Premium genauso gilt und wodurch „aufeinanderfolgend“ unterbrochen wird, wurde nicht ausdrücklich bestätigt (Fragen 4, 5).
- **Zeitfenster-Grenzfälle:** Was mit einem 30-Minuten-Angebot passiert, wenn währenddessen die Zwei-Stunden-Grenze erreicht wird; ob nach regulärem Buchungsschluss noch automatisch nachgerückt werden darf (Fragen 8, 9, 11).
- **Basic-Kontingent:** Zuordnung zum Monat des Buchungszeitpunkts oder des Kurstermins bleibt unklar; *empfohlene Annahme:* Monat des Kurstermins (Frage 11).
- **Bestandsdaten:** Aktuelle Mitglieds- und Tarifdaten sollen importiert werden; lückenhafte alte Buchungen nicht. Frühere Tarifwechsel und Pausen sind teilweise nicht rekonstruierbar (Frage 16).
- **Plattform:** Mobile Nutzung ist Pflicht. Native App vs. mobile Web-App/PWA wurde Lisa bewusst zur technischen Entscheidung überlassen (Frage 13).
- **Version 2 – Coaching:** Premium-Sonderregel bei bezahlten Coaching-Stornierungen, Zahlungsanbieter und Buchhaltungsintegration fehlen (Fragen 17, 18).
- **Projektbedingungen:** Budget und Fertigstellungstermin nicht festgelegt, kein Pflichtanschluss an bestehende Software (Frage 20).


