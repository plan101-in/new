# TerraScope AI — manual backend repair

This pack is based on `terrascope-ai (1).zip` uploaded on 2026-11-02.

It contains source-level repairs for your existing React / Vite app. It does not
alter the layout, CSS, model URL, or satellite image assets.

## 1. Make a backup before pasting

Export your current Google AI Studio project once. Keep your existing Google
Colab notebook and GitHub Pages model unchanged.

## 2. Database migration (required)

In Supabase > SQL Editor, execute `BACKEND_MIGRATION.sql`. It assumes you have
already run the initial four-table setup and the `terrascope-images` private
bucket policies. If those resources do not exist, run the initial setup SQL
from the previous instructions first. Do not run a schema reset.

The migration adds `terrascope_predictions.in_review_queue`. Previously,
`Clear Queue` could delete prediction history. The new code only unqueues rows.

## 3. AI Studio environment variables (required)

Configure these exact browser-build variable names:

```
VITE_SUPABASE_URL=https://YOUR_PROJECT.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=sb_publishable_YOUR_KEY
```

Older publishable/anon keys can also use `VITE_SUPABASE_ANON_KEY`.
Never use `service_role`, `sb_secret_...`, or the database password in browser
code. Rebuild after changing environment variables.

If your AI Studio environment stores the variables only on the server, ensure
they are explicitly supplied to Vite during the client build; Vite's
`import.meta.env.VITE_*` reads build-time values.

## 4. Manually replace these files

Copy the complete matching source file contents from this patch pack into the
same path inside AI Studio's Code tab. You can do it without an AI Studio prompt:

1. `src/services/supabaseClient.ts`
2. `src/services/supabaseStorageService.ts`
3. `src/services/predictionService.ts`
4. `src/services/tfliteService.ts`
5. `src/App.tsx`
6. `src/components/ClassificationView.tsx`
7. `src/components/ReviewQueueView.tsx`
8. `src/components/EvaluationView.tsx`
9. `src/components/EvaluationImportModal.tsx`
10. `src/components/AuthModal.tsx`
11. `package.json` (fixes the Vite 8 / esbuild peer dependency conflict)

Do **not** upload the original `.env` from the ZIP or overwrite other files.
This patch pack intentionally does not contain credentials, original images,
or model binaries. The GitHub Pages model remains external.

## 5. Manual end-to-end tests

1. Sign in using the Supabase account you created.
2. Classify a JPG and check the `terrascope_predictions` table for one new row.
3. Check `Storage > terrascope-images` for its private image.
4. Refresh and confirm the saved classification/history is still available.
5. Add that prediction to the real Review Queue and confirm
   `in_review_queue = true` for the row.
6. Approve or reclassify it and confirm a `terrascope_review_events` row.
7. Remove it from the queue; confirm `in_review_queue = false` but the
   prediction row still exists.
8. Import evaluation JSON once; confirm it appears in
   `terrascope_evaluations` and persists after refresh.
9. Log out and back in; re-check saved results.
10. Export a JSON backup. Note that image binaries are NOT contained in the
    JSON file; keep originals in private Supabase Storage or back them up
    separately to Google Drive.

## 6. What was actually verified

- `npm run lint` passed (TypeScript type checking).
- `npm run build` passed (Vite production build).
- No live Supabase API writes, login, image retrieval, or RLS tests were run
  here. Complete Step 5 before treating the backend as operational.

## 7. Important limitations

- Supabase Free can pause after inactivity, and automatic backups are not
  included. Export important data periodically.
- Example scores are still illustrative and should not be reported as real
  competition metrics.
- Your current TFLite weights remain trained on synthetic practice data until
  you retrain with the organizer's official dataset.
- Files larger than about 6 MB may benefit from resumable storage uploads.
- A private file's signed URL expires; the app must request a fresh URL for
  future visits.
