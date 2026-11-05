# RedNetz

Deine Infrastruktur an einer Stelle: Server, Domains, Docker und Dienste, Monitoring, Netzwerkkarte und Logs. Kostenlos, selbst gehostet.

```bash
curl -fsSL https://raw.githubusercontent.com/theredstonee/rednetz-docker/main/install.sh | sudo bash
```

Update: `curl -fsSL https://raw.githubusercontent.com/theredstonee/rednetz-docker/main/update.sh | sudo bash`

Entfernen: `sudo bash install.sh --deinstallieren`

Die App hört nur intern oder auf 127.0.0.1. Nach außen geht es über Caddy, deinen Webserver oder einen Cloudflare Tunnel.

## Agent

Unter „Server anbinden“ bekommst du einen Einzeiler mit Einmal-Code. Der Agent baut nur ausgehende Verbindungen auf und öffnet keinen Port.

Er liest Betriebssystem, Hardware, Netz, offene Ports mit Prozessname, Dienste, offene Updates, Backup-Timer und Docker über eine feste Liste lesender Aufrufe. Aus nginx, Apache, Caddy und HAProxy nimmt er nur Servernamen und Ziele, aus Traefik nur Regel und Port, aus Nginx Proxy Manager nur Namen und Ziele der erzeugten `proxy_host`-Dateien. Was der Agent nicht lesen darf, fehlt dann in der Karte. Erkannte Namen kommen als Domain ins Inventar. Er liest nie Umgebungsvariablen, Befehlszeilen oder Compose-Dateien. Logs nur, wenn du eine Quelle einschaltest.

Wer den Docker-Socket lesen darf, hat auf dem Server praktisch root-Rechte. Du kannst den Docker-Zugriff je Server abschalten (`--ohne-docker`), dann fehlen die Docker-Daten.

Podman liest der Agent über dieselben Aufrufe, wenn es kein Docker gibt. Dafür beim Einzeiler `--mit-podman`: Das gibt den Podman-Socket des Systems für den Agent frei und entspricht ebenfalls praktisch root-Rechten. Rootless-Podman ist nicht abgedeckt.

Updates des Agents werden nur mit gültiger Signatur eingespielt.

## Cloudflare (Beta)

Lief bisher nur gegen einen Nachbau aus der Cloudflare-Doku, nie gegen das echte Cloudflare. Kann scheitern oder unvollständige Werte zeigen.

Unter „Anbindungen“ einen API-Token mit **Zone Read** und **DNS Read** eintragen. RedNetz ruft nur lesend ab und zeigt Zonen und Einträge zentral. Hat der Token Schreibrechte und Cloudflare gibt das preis, warnt RedNetz.

## Plesk (Beta)

Lief bisher nur gegen einen Nachbau aus der Plesk-Doku, nie gegen ein echtes Plesk. Kann scheitern oder unvollständige Werte zeigen.

Normales Konto: Abfragen über die XML-API. Administrator: zusätzlich REST. Jede Abfrage wird einzeln gemessen, fehlende Rechte stehen offen da, nichts wird geraten. Was fehlt, ergänzt du von Hand.

## Monitoring

Uptime, CPU, RAM, Platte, Container, Websites, Dienste und Ports. Ein Ausfall zählt erst nach fehlendem Lebenszeichen und einer Gegenprobe. Meldungen in der App und per E-Mail, mit Hysterese, einmal zu Beginn und einmal am Ende.

Rohdaten bleiben 7 Tage, 5-Minuten-Werte 30 Tage, Stundenwerte 1 Jahr.

**Bekannte Lücken** sind abschaltbar und standardmäßig aus. Eingeschaltet schickt der Agent die Paketliste an deine RedNetz-Instanz, diese fragt [OSV.dev](https://osv.dev) mit Paketname, Version und Ökosystem. Hostname und IP gehen nicht mit.

**SMART** braucht root und ist darum optional (`--mit-smart`): Ein eigener Timer schreibt nur die Ausgabe von `smartctl` in eine Datei, der Agent bleibt ohne Rechte. Nur Anzeige.

**Backups** erkennt RedNetz an Timern und Containern, eigene Skripte melden sich über `POST /api/v1/backups/melden` (Bereich `backups:melden`).

## Log-Kollektor

Alle Quellen sind aus, bis du sie einzeln einschaltest: journald, Dateien aus einer festen Liste, Docker. `--ohne-logs` schaltet den Kollektor auf einem Server fest ab.

Der Agent maskiert Passwörter, Tokens, Schlüssel und Anmeldedaten in Adressen vor dem Versand. **Das erkennt nicht alles.** Logs können personenbezogene Daten enthalten: Prüfe, was du einschaltest, und lege eine passende Aufbewahrung fest (je Quelle in Tagen). Lesen darf nur, wer das Recht `logs.lesen` hat, jeder Abruf steht im Protokoll.

Eine Tagesgrenze je Server (Standard 500 MB) schützt die Datenbank. Wird sie erreicht, warnt RedNetz und verwirft den Rest des Tages.

## Andere Produkte

**Anmeldung über RedClub:** Unter „Anbindungen“ die Adresse, Client-ID und Geheimnis eintragen, in RedClub als Weiterleitung `https://<RedNetz>/api/oidc/rueck`. Das Notfall-Konto mit Passwort bleibt immer nutzbar.

**RedClub** liest über `GET /api/produkt/v1/stand` (Bereich `produkt:lesen`), **RedDrop** erscheint als Gruppe und auf der Karte. Geteilt wird nur der Zustand, nie Dateinamen, Nutzerdaten, Ports oder Zugangsdaten. Aktionen laufen über einen getrennten Token und sind standardmäßig aus.

**Router:** Bietet ein Router auf dem Gerät des Agents eine lokale Schnittstelle an, liest der Agent sie. Einmalig als Benutzer des Agents: `sudo -u rednetz-agent env REDNETZ_KONFIG=/var/lib/rednetz-agent/agent.json /var/lib/rednetz-agent/bin/rednetz-agent router --url http://127.0.0.1:PORT --token-datei DATEI`. Firewall, Netze und Weiterleitungen sind nur Anzeige.
