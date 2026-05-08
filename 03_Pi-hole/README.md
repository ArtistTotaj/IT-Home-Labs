# Pi-hole – Netzwerkweiter DNS-Adblocker auf Raspberry Pi

Heimlabor-Projekt: Aufbau eines DNS-basierten Adblockers für das gesamte Heimnetzwerk.  
Ziel: Werbung und Tracking auf allen Geräten blockieren – ohne clientseitige Software.

---

## Technologie-Stack

| Komponente | Details |
|---|---|
| Hardware | Raspberry Pi (AArch64 / 64-bit ARM) |
| Betriebssystem | Raspberry Pi OS Lite 64-bit (Debian Trixie) |
| Software | Pi-hole Core v6.4.2 / Web v6.5 |
| Router | Sagemcom Fibre X6 |
| Upstream-DNS | Cloudflare 1.1.1.1 mit DNSSEC |
| Blocklist | StevenBlack Unified Hosts List |
| Remote-Zugriff | SSH (Passwort-Authentifizierung) |
| Host-System | macOS / zsh |

---

## Projektziel

Alle DNS-Anfragen im Heimnetzwerk laufen über den Raspberry Pi. Pi-hole filtert bekannte Werbe- und Tracking-Domains direkt auf DNS-Ebene heraus – bevor eine Verbindung aufgebaut wird. Das schützt alle Geräte im Netzwerk gleichzeitig: Smartphones, Smart-TVs, IoT-Geräte.

---

## Umsetzung

### Schritt 1 – OS flashen mit Raspberry Pi Imager

Raspberry Pi OS Lite (64-bit) auf SD-Karte geschrieben.  
Konfiguration direkt im Imager unter *Anpassung*:

- Benutzername gesetzt
- SSH mit Passwort-Authentifizierung aktiviert
- WLAN-Zugangsdaten hinterlegt
- Hostname: `pihole`

**Warum Lite?** Kein Desktop-Environment nötig – spart Ressourcen, kleinere Angriffsfläche.

---

### Schritt 2 – SSH-Verbindung aufbauen

```bash
ssh pishakel@192.168.1.167
```

**Problem aufgetreten:** SSH verweigerte Verbindung mit `Host key verification failed`.

**Ursache:** Nach dem Neuflashen der SD-Karte ändert sich der SSH-Host-Key des Pi.  
macOS hatte den alten Fingerprint in `~/.ssh/known_hosts` gespeichert und blockierte die Verbindung – korrektes Sicherheitsverhalten zum Schutz vor Man-in-the-Middle-Angriffen.

**Lösung:**
```bash
ssh-keygen -R 192.168.1.167
ssh pishakel@192.168.1.167
```

Der alte Eintrag wird aus `known_hosts` entfernt, beim nächsten Verbinden wird der neue Fingerprint akzeptiert und gespeichert.

---

### Schritt 3 – Pi-hole installieren

```bash
curl -sSL https://install.pi-hole.net | bash
```

**Installer-Output (Auszug):**
```
[✓] Detected AArch64 (64 Bit ARM) architecture
[✓] Upstream DNS: Cloudflare (DNSSEC) (1.1.1.1, 1.0.0.1)
[✓] Installing StevenBlack's Unified Hosts List
[i] Number of gravity domains: 80799 (80799 unique domains)
[✓] DNS resolution is available
[✓] Done.
```

80.799 Domains werden blockiert – automatisch, für alle Geräte im Netzwerk.

---

### Schritt 4 – DNS-Integration im Router

Der Sagemcom Fibre X6 bietet kein direktes DNS-Server-Feld in den LAN-Einstellungen.  
Lösung: Pi-hole übernimmt den DHCP-Server vom Router.

**Vorgehen:**
1. DHCP im Router deaktivieren (LAN IP → DHCP → Aus)
2. DHCP in Pi-hole aktivieren: Admin-Interface → Settings → DHCP
3. Pi-hole verteilt nun an alle Geräte automatisch sich selbst als DNS-Server

---

## Was dabei gelernt wurde

**SSH & Public-Key-Infrastruktur**
- Wie SSH-Host-Keys funktionieren und warum sie sich ändern
- `known_hosts` – Zweck und manuelle Verwaltung
- Unterschied Passwort-Authentifizierung vs. Public-Key-Authentifizierung

**Linux-Grundlagen**
- Navigation und Dateieditierung im Terminal (`nano`, `systemctl`)
- SSH-Daemon-Konfiguration (`/etc/ssh/sshd_config`)
- Service-Management mit `systemctl restart`

**Netzwerk & DNS**
- Wie DNS-Auflösung funktioniert
- DHCP-Server-Rolle und warum nur ein DHCP-Server pro Netzwerk laufen darf
- DNS-over-HTTPS / DNSSEC mit Cloudflare
- Netzwerk-Traffic-Analyse über Pi-hole Admin-Interface

**Sicherheitskonzepte**
- Man-in-the-Middle-Angriffe und wie SSH davor schützt
- Warum Host-Key-Verification ein Sicherheitsmerkmal ist, kein Bug
- Netzwerksegmentierung durch DNS-Filterung

---

## Ergebnis

- Pi-hole läuft stabil auf `192.168.1.167`
- 80.799 Domains blockiert
- Admin-Interface erreichbar unter `http://192.168.1.167/admin`
- Alle Netzwerkgeräte nutzen Pi-hole als DNS-Resolver
- Upstream-DNS: Cloudflare mit DNSSEC-Validierung

---

## Nächste Schritte

- [x] Statische IP für den Pi im Router reservieren (IP-Reservierung via MAC-Adresse)
- [x] Zusätzliche Blocklisten hinzufügen
- [x] SSH auf Public-Key-Authentifizierung umstellen (Passwort-Login deaktivieren)
- [ ] Pi-hole Monitoring einrichten (Uptime, blocked queries)
- [ ] Unbound als lokalen DNS-Resolver aufsetzen (kein Upstream-Provider mehr nötig)

---

## Referenzen

- [Pi-hole Dokumentation](https://docs.pi-hole.net)
- [StevenBlack Hosts](https://github.com/StevenBlack/hosts)
- [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
