# TestCloneRepo

## Bidirectional Sync: GitHub ↔ GitLab

Dieses Repository ist so konfiguriert, dass alle Commits automatisch zwischen
diesem GitHub-Repo und dem privaten GitLab-Server synchronisiert werden.

| Richtung | Auslöser | Workflow |
|---|---|---|
| GitHub → GitLab | Jeder `git push` auf GitHub | `sync-to-gitlab.yml` |
| GitLab → GitHub | Automatisch alle 5 Minuten + manuell | `sync-from-gitlab.yml` |

---

## Einrichtung (einmalig)

### 1. SSH-Schlüsselpaar erstellen

```bash
ssh-keygen -t ed25519 -C "github-gitlab-sync" -f gitlab_sync_key -N ""
```

Damit entstehen zwei Dateien:
- `gitlab_sync_key` → **privater Schlüssel** (kommt nach GitHub)
- `gitlab_sync_key.pub` → **öffentlicher Schlüssel** (kommt nach GitLab)

### 2. Öffentlichen Schlüssel in GitLab hinterlegen

Gehe in deinem GitLab-Projekt zu  
**Settings → Repository → Deploy keys**  
und füge den Inhalt von `gitlab_sync_key.pub` als Deploy Key mit
**Schreibzugriff (Write access)** ein.

### 3. GitHub Secrets setzen

Gehe in diesem GitHub-Repository zu  
**Settings → Secrets and variables → Actions → New repository secret**  
und lege folgende Secrets an:

| Secret | Wert |
|---|---|
| `GITLAB_SSH_KEY` | Inhalt der Datei `gitlab_sync_key` (privater Schlüssel, inkl. `-----BEGIN ...`) |
| `GITLAB_HOST` | Hostname deines GitLab-Servers (z. B. `gitlab.example.com`) |
| `GITLAB_REPO_PATH` | Pfad des Repositories auf GitLab (z. B. `meinuser/meinprojekt`) |
| `GITLAB_KNOWN_HOSTS` | Host-Key-Eintrag für den GitLab-Server – lokal erzeugen mit: `ssh-keyscan -H gitlab.example.com` |

### 4. Workflow-Berechtigungen prüfen

Gehe zu **Settings → Actions → General → Workflow permissions**  
und stelle sicher, dass **„Read and write permissions"** aktiviert ist.
Dies erlaubt dem `sync-from-gitlab.yml`-Workflow, mit `GITHUB_TOKEN` in das
GitHub-Repository zu pushen.

---

## Wie funktioniert die Synchronisierung?

- **GitHub → GitLab**: Bei jedem Push auf GitHub wird der Workflow
  `sync-to-gitlab.yml` ausgelöst, der alle Branches und Tags per SSH an den
  GitLab-Server spiegelt.

- **GitLab → GitHub**: Der Workflow `sync-from-gitlab.yml` läuft alle
  5 Minuten, holt alle Änderungen von GitLab und pusht sie nach GitHub.
  Da der Workflow dabei `GITHUB_TOKEN` nutzt, werden durch diesen Push keine
  weiteren Workflows ausgelöst – eine Endlosschleife wird so verhindert.

- **Manuelle Synchronisierung**: Der `sync-from-gitlab.yml`-Workflow kann
  jederzeit manuell unter **Actions → Sync GitLab → GitHub → Run workflow**
  gestartet werden.