# KamelPay Compliance Academy — Training Site

Static web app for compliance training with a Firebase-backed admin dashboard.

## Files

| File | Purpose |
|------|---------|
| `index.html` | Learner-facing training + quiz page. Records progress/results to Firestore. This is the **training link** shared with users. |
| `admin/index.html` | Admin dashboard (read-only). Email-link sign-in, allowlisted admins only. Displays attempts and starts. |

## Backend

- **Platform:** Firebase (Firestore + Auth, email-link sign-in)
- **Project ID:** `kamelpay-compliance-academy`
- **Firebase config:** hardcoded in both HTML files (public client config — this is normal for Firebase web apps; real protection is Firestore Security Rules).

### Firestore collections

| Collection | Doc ID | Written by | Fields |
|------------|--------|-----------|--------|
| `starts` | sanitized learner email | `index.html` when a learner begins | `name`, `email`, `lastStarted` (timestamp) |
| `attempts` | attempt ID | `index.html` on quiz submit | `name`, `email`, `score`, `total`, `percentage`, `passed`, `certId` (if passed), `timestamp` |

Pass mark: **80%**. Admin allowlist (in `admin/index.html`, `ADMIN_EMAILS`): `syeda.zeenat@kamelpay.com`, `ramesh.kumar@kamelpay.com`.

## Common admin action: clear all training data (fresh start)

The admin dashboard is **read-only** — it has no delete button. To wipe all records before a
fresh round of training, delete every document in both collections. This is **irreversible**.

### Using the Firebase CLI

```bash
# One-time: install + log in (must be Owner/Editor on the GCP project)
npm install -g firebase-tools
firebase login

# Delete both collections (recursive). Prompts for confirmation.
firebase firestore:delete attempts --recursive --project kamelpay-compliance-academy
firebase firestore:delete starts   --recursive --project kamelpay-compliance-academy
```

Add `--force` to skip the interactive confirmation (use with care).

### Optional backup before deleting

```bash
# Requires a service-account key + a GCS bucket; exports to Cloud Storage
gcloud firestore export gs://<your-bucket>/backups/$(date +%F) \
  --collection-ids=attempts,starts --project kamelpay-compliance-academy
```

## Notes

- After clearing, the admin dashboard will show empty stats until learners use the fresh training link again.
- The Firestore Security Rules (not in this repo) govern who can read/write/delete — CLI deletion runs with the authenticated user's IAM permissions, bypassing client-side rules.
