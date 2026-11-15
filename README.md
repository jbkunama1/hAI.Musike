# hAI.Musike

Musike als Docker-/Portainer-Stack für das externe Docker-Netzwerk `highfishNetwork`.

Upstream: https://github.com/Ernani1234/musike
Image: `ghcr.io/jbkunama1/hai.musike:latest`
Projektseite: [index.html](index.html)

## Enthalten

- Original-Musike-Weboberfläche und Server, gebaut mit dem Upstream-Dockerfile.
- Multi-Arch-Image für Linux amd64 und arm64.
- Persistente Volumes für Bibliothek und Videocache.
- Upstream-Healthcheck, Non-root-Laufzeit, rotierende Containerlogs.
- Täglicher vollständiger Neubau mit aktuellem Upstream-Quellcode und frischen Basisimages/Build-Abhängigkeiten.
- Separate statische Projektseite; sie ersetzt nicht die Musike-Weboberfläche.

## Installation mit Portainer

1. Den ersten erfolgreichen Workflow unter Actions abwarten.
2. In GitHub beim GHCR-Paket die Sichtbarkeit prüfen. Für anonymes Pull muss das Paket öffentlich sein; sonst in Portainer passende GHCR-Zugangsdaten hinterlegen. Ein öffentliches Repository garantiert kein öffentliches Paket.
3. Das Netzwerk `highfishNetwork` muss auf dem gewählten Docker-Endpunkt bereits existieren.
4. In Portainer einen Stack `hai-musike` anlegen und `docker-compose.yml` einfügen, alternativ aus diesem Repository deployen.
5. Optional Stack-Variable `MUSIKE_PORT` setzen, falls Hostport 4180 belegt ist.
6. Stack deployen und `http://SERVER-IP:4180` öffnen.

Im selben Docker-Netzwerk erreicht ein Reverse Proxy die Anwendung unter `http://hai-musike:4180`. HTTPS und Zugriffsschutz am Proxy konfigurieren; nicht ungeschützt ins Internet stellen. Für reinen Proxybetrieb kann das `ports`-Mapping entfernt werden.

## Installation per CLI

```bash
git clone https://github.com/jbkunama1/hAI.Musike.git
cd hAI.Musike
docker network inspect highfishNetwork
# Nur falls das Netzwerk noch fehlt:
# docker network create highfishNetwork
docker compose up -d
```

## Tägliche Updates

Der Workflow läuft täglich um 03:30 UTC (04:30 MEZ / 05:30 MESZ), zusätzlich bei Workflowänderungen und manuell per Actions > Build and publish Musike > Run workflow. GitHub kann geplante Läufe verzögern. Geplante Workflows öffentlicher Repositories können nach längerer Inaktivität deaktiviert werden; Actions im Blick behalten.

Jeder Lauf checkt den aktuellen Default-Branch von Ernani1234/musike aus und baut dessen Dockerfile ohne Buildcache. Das übernimmt auch Änderungen am Upstream-Dockerfile. Das ist bewusst ein bewegliches Upstream-Ziel und kein getesteter stabiler Releasekanal. Den verwendeten Commit dokumentieren Image-Label, SHA-Tag und Workflow-Zusammenfassung.

Tags: `latest`, `upstream-<vollständige SHA>`, `build-<YYYYMMDD>-<Laufnummer>`. Auch derselbe Quellcommit kann durch aktualisierte Abhängigkeiten ein anderes Image ergeben. Für einen exakt fixierten Stand den Image-Digest verwenden.

Ein täglicher Image-Build aktualisiert KEINEN laufenden Container. Serverupdate:

```bash
docker compose pull
docker compose up -d
docker compose ps
```

In Portainer den Stack mit erneutem Image-Pull aktualisieren. Ein automatisches Server-Redeployment oder Docker-Socket-Zugriff wird hier absichtlich nicht eingerichtet.

## Daten und Backup

| Volume | Containerpfad | Inhalt |
|---|---|---|
| musike-data | /data | Bibliothek, Playlists, Favoriten und Verlauf |
| musike-cache | /cache | Videocache, standardmäßig maximal 2048 MB |

Compose/Portainer versieht Volumenamen normalerweise mit dem Stack-/Projektnamen. Vor Updates Bibliothek in Musike über Settings > Your data > Export sichern. Für Dateisicherungen den Container stoppen und das Datenvolume sichern. `docker compose down -v` löscht die Volumes und ist kein Updatebefehl.

## Konfiguration und Fehlerdiagnose

- Region ist auf `DE` gesetzt. Die Musike-Oberfläche bleibt in den von Upstream unterstützten Sprachen; dies fügt keine deutsche Übersetzung hinzu.
- Cachelimit: `MUSIKE_CACHE_MAX_MB` im Stack anpassen.
- Healthcheck: `/api/health` auf internem Port 4180 (im Upstream-Dockerfile definiert).
- Logs: `docker compose logs --tail=100 musike`.
- Wiedergabeprobleme: yt-dlp in Musike unter Settings aktualisieren oder ein frisches Image ziehen. Manuelle Toolupdates im Container gehen bei Neuerstellung verloren.
- YouTube kann Rechenzentrums-/VPN-IP-Adressen blockieren; Upstream empfiehlt einen Heimserver.
- Bei `network highfishNetwork declared as external, but could not be found`: Netzwerk auf genau diesem Docker-Endpunkt prüfen.
- Bei `denied` beim Image-Pull: erfolgreichen Build, Paketname und GHCR-Paketsichtbarkeit bzw. Registry-Anmeldung prüfen.

## Indexseite

`index.html` ist eine eigenständige, mobilfreundliche Dokumentations-/Projektseite. Lokal öffnen oder über einen statischen Webserver bereitstellen. GitHub Pages wird durch diese Dateien nicht automatisch aktiviert.

## Herkunft und Rechte

Musike stammt von Ernani1234 und steht laut Upstream unter MIT-Lizenz. Der Build verwendet den Original-Quellbaum einschließlich der dortigen Lizenzdateien; dieses Repository enthält Deploymentkonfiguration, keine umbenannte Eigenentwicklung des Players. Upstream-Lizenz und rechtliche Hinweise: https://github.com/Ernani1234/musike#license und https://github.com/Ernani1234/musike#legal-notice. Beachte die Nutzungsbedingungen der Inhaltsanbieter und nutze nur Inhalte, für die du die erforderlichen Rechte hast.

## Prüfstatus

Die Konfiguration ist vorbereitet. Ein erfolgreicher GitHub-Actions-Build und ein Wiedergabetest auf deinem Docker-Host müssen noch bestätigt werden.
