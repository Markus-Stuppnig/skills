---
name: zukunftssicher-definition-of-done
description: Definition of Done für Zukunftssicher – verbindliche Checkliste für Tickets, Features, Bugfixes und Projekte. Sollte bei jedem abzuschließenden Arbeitsschritt geladen und abgehakt werden.
---

# Definition of Done – Zukunftssicher

Gilt für **alle** Arbeiten: Tickets, Features, Bugfixes, kleine Änderungen, ganze Projekte. Keine Ausnahmen.

Ein Ticket/Feature/Projekt ist erst **Done**, wenn **alle** Punkte erfüllt sind.

---

## 1. UX (wichtigster Punkt)
- [ ] User Flow durchgespielt – funktioniert auf Anhieb, ohne Nachdenken
- [ ] Mobil getestet (mindestens iPhone + Android)
- [ ] Desktop getestet (Chrome + Safari + Firefox)
- [ ] Ladezeiten akzeptabel (< 3 Sek. für Websites)
- [ ] Fehlermeldungen sind verständlich (keine Technik-Codes für Endnutzer)
- [ ] Wichtige Aktionen haben Feedback (Loading, Success, Error)

## 2. Dokumentation
- [ ] Code-Kommentare bei nicht-offensichtlicher Logik
- [ ] README / Kundendokumentation aktualisiert
- [ ] Setup-Anleitung vorhanden (wie starten, wo liegt was)
- [ ] Bei Automatisierungen: Flow dokumentiert (Input → Prozess → Output)
- [ ] Änderungen seit letztem Release festgehalten
- [ ] Kunde versteht die Doku ohne Rückfrage

## 3. DSGVO
- [ ] Datenverarbeitung dokumentiert
- [ ] Cookie-Banner (wenn nötig) korrekt eingebunden
- [ ] Datenschutzerklärung erwähnt relevante Tools/Dienste
- [ ] Daten werden in EU verarbeitet, wenn möglich (Hetzner: ✅)
- [ ] Keine Tracking-Tools ohne Einwilligung

## 4. Security
- [ ] Keine Passwörter/Keys im Code oder Repo
- [ ] Environment Variables für Secrets
- [ ] HTTPS überall
- [ ] Dependencies aktuell (keine bekannten Lücken)
- [ ] Input-Validierung bei Forms und APIs
- [ ] Zugriffsrechte geprüft (nur wer muss, darf rein)

## 5. Staging (Pflicht)
- [ ] Auf Staging-Umgebung deployed
- [ ] Auf Staging getestet (nicht nur lokal)
- [ ] Kunde hatte Möglichkeit zur Review auf Staging
- **Ausnahme:** Nur bei trivialen, risikolosen Änderungen (z. B. Typo-Fix) – dokumentieren, warum keine Staging-Runde

## 6. Deployment
- [ ] Deployment erfolgt durch Markus (nicht durch Kunden)
- [ ] Backup vor Deployment (bei bestehenden Systemen)
- [ ] Deployment-Zeitpunkt mit Kunden abgestimmt
- [ ] Rollback-Plan klar (wie zurück, falls was schiefgeht)
- [ ] Nach Deployment: Live-Check (funktioniert alles in Production?)

## 7. Abnahme (bei Projektabschluss / größeren Features)
- [ ] Abnahme **in Person** mit Kunden
- [ ] Funktionen gemeinsam durchgegangen
- [ ] Offene Punkte schriftlich festgehalten
- [ ] Kunde gibt Abnahme aktiv frei (nicht stillschweigend)

## 8. Übergabe (bei Projektabschluss)
- [ ] Vollständige Dokumentation übergeben
- [ ] Einschulung durchgeführt (Kunde kann selbst mit dem System arbeiten)
- [ ] Zugangsdaten sicher übergeben (Passwort-Manager, nicht per Mail/Chat)
- [ ] Supportphase besprochen (Umfang, Dauer, Kontaktweg)

---

## Definition of "Not Done"
Ein Ticket ist **nicht** done, wenn:
- "Funktioniert bei mir lokal" – aber nicht auf Staging
- Dokumentation fehlt oder ist unverständlich
- UX wurde nicht durchgespielt
- Kunde hat nichts gesehen/abgenommen
- Secrets liegen im Code

## Kurzform (für kleine Tickets)
Bei Mini-Änderungen reicht mental diese Kurzform:
1. **UX** passt
2. **Doku** aktualisiert
3. **Staging** getestet
4. **Sicher** (keine Secrets leaked, DSGVO ok)
5. **Deployed** durch mich
