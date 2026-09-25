# Nurse WOYZ

Nurse WOYZ is a standalone clone of the WOYZ notes app, adapted for nurse-facing clinical documentation. It keeps the same three-page structure:

- `index.html`: user page for nurse note creation and daily patient workflows.
- `admin.html`: admin/worklist page for mapped nurse groups.
- `master-admin.html`: master admin page for creating groups and assigning admin access.

The project is intentionally disconnected from the original WOYZ Firebase project, email backend, CNAME, and GitHub repository.

## Firebase Data Model

Notes are stored at:

```text
users/{firebaseAuthUid}/notes/{noteId}
```

Each signed-in Firebase Authentication user owns their notes. Admin and group access is controlled through Firestore documents and `firestore.rules`.

## Create New Infrastructure

Use `setup-new-project.sh` only from a standalone copy of this folder that is not inside another Git repository. The script creates:

- a new GitHub repository named `Nurse-Woyz-Aster`;
- a new Firebase project with display name `Nurse WOYZ`;
- a new default Firestore database;
- a new Firebase Web app;
- a new `.firebaserc`;
- a first Git commit and push.

Run it with the new master admin email:

```bash
MASTER_ADMIN_EMAIL="master@example.com" ./setup-new-project.sh
```

Optional overrides:

```bash
GITHUB_VISIBILITY=public FIRESTORE_LOCATION=asia-south1 MASTER_ADMIN_EMAIL="master@example.com" ./setup-new-project.sh
```

The script refuses to reuse an existing GitHub repository or Firebase project.

## Manual Firebase Steps

After the script completes:

1. Enable **Email/Password** in Firebase Authentication.
2. Create the master admin account using the same email passed as `MASTER_ADMIN_EMAIL`.
3. Create nurse/user accounts in Firebase Authentication.
4. If using GitHub Pages, add the GitHub Pages hostname to Authentication authorized domains.
5. Enable GitHub Pages or deploy with Firebase Hosting.

## Email Sending

Email UI remains in the app, but sending is disabled because the original cloud function belonged to the old project. Configure a new Nurse WOYZ email backend before enabling `EMAIL_FUNCTION_URL` and `EMAIL_BACKEND_CONFIG` in `index.html`.

## Checks

Before publishing after edits, run:

```bash
python3 - <<'PY'
from pathlib import Path
for name,out in [('admin.html','/tmp/nurse-woyz-admin-module.js'),('index.html','/tmp/nurse-woyz-index-module.js'),('master-admin.html','/tmp/nurse-woyz-master-module.js')]:
    html = Path(name).read_text()
    start = html.index('<script type="module">') + len('<script type="module">')
    end = html.index('</script>', start)
    Path(out).write_text(html[start:end])
PY
node --check /tmp/nurse-woyz-index-module.js
node --check /tmp/nurse-woyz-admin-module.js
node --check /tmp/nurse-woyz-master-module.js
node --check sw.js
```
