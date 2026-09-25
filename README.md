# Nurse WOYZ

Nurse WOYZ is a standalone clone of the WOYZ notes app, adapted for nurse-facing clinical documentation. It keeps the same three-page structure:

- `index.html`: user page for nurse note creation and daily patient workflows.
- `admin.html`: admin/worklist page for mapped nurse groups.
- `master-admin.html`: master admin page for creating groups and assigning admin access.

The project is intentionally disconnected from the original WOYZ Firebase project, email backend, CNAME, and old GitHub repositories. It is currently pointed at the renamed Firebase project **Nurse WOYZ Aster** with project ID `rajagiri-neurology`.

## Firebase Data Model

Notes are stored at:

```text
users/{firebaseAuthUid}/notes/{noteId}
```

Each signed-in Firebase Authentication user owns their notes. Admin and group access is controlled through Firestore documents and `firestore.rules`.

## Create New Infrastructure

The app is already configured for the existing Firebase project `rajagiri-neurology`, whose display name has been renamed to **Nurse WOYZ Aster**. Firestore and Authentication can be configured later.

Master admin access is controlled by private Firestore documents at `masterAdmins/{firebaseAuthUid}`. Do not hardcode master-admin emails or UIDs into public client files.

Use `setup-new-project.sh` only if you later decide to create a completely separate Firebase project and GitHub repository from a standalone copy of this folder that is not inside another Git repository. The script creates:

- a new GitHub repository named `Nurse-Woyz-Aster`;
- a new Firebase project with display name `Nurse WOYZ`;
- a new default Firestore database;
- a new Firebase Web app;
- a new `.firebaserc`;
- a first Git commit and push.

Run it from the project folder:

```bash
./setup-new-project.sh
```

Optional overrides:

```bash
GITHUB_VISIBILITY=public FIRESTORE_LOCATION=asia-south1 ./setup-new-project.sh
```

The script refuses to reuse an existing GitHub repository or Firebase project.

## Manual Firebase Steps

After the script completes:

1. Enable **Email/Password** in Firebase Authentication.
2. Create the master admin account in Firebase Authentication.
3. Create a private Firestore marker document at `masterAdmins/{firebaseAuthUid}` for the master admin user.
4. Create nurse/user accounts in Firebase Authentication.
5. If using GitHub Pages, add the GitHub Pages hostname to Authentication authorized domains.
6. Enable GitHub Pages or deploy with Firebase Hosting.

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
