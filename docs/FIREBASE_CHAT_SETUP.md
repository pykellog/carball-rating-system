# Firebase Chat — Security Rules, Rate-Limit & Deployment

This file explains the Realtime Database security rules, simple rate-limiting, and steps to test and commit changes to GitHub.

Files added:
- `docs/firebase-rules.json` — recommended Realtime Database rules

1) Deploying security rules

Option A — Firebase Console (GUI):
- Go to Realtime Database → Rules, paste the contents of `docs/firebase-rules.json`, and publish.

Option B — Firebase CLI:
- Install Firebase CLI and login:
```bash
npm install -g firebase-tools
firebase login
```
- Then set rules (from repo root):
```bash
firebase database:rules:set docs/firebase-rules.json --project carball-chat-system
```

2) What the rules do (summary)
- `messages`: anyone can read, only authenticated users (`auth != null`) can write.
- `messages/$msg` validation checks presence of required fields, limits text length and displayName length, ensures timestamp is numeric and reasonably close to server time, and uses `presence/auth.uid/lastMsgTs` for a simple server-side rate-check (requires client to update that field when sending).
- `presence`: read access is public, but write access is restricted under `presence/$uid` so only `auth.uid === $uid` may modify its own record.
- `presence/$uid` validation ensures the user record contains `displayName` and `ts`, with displayName length limits and numeric timestamp.

3) Client-side changes (already applied)
- Client enforces a 3s rate-limit and updates `presence/<uid>/lastMsgTs` when a message is sent. This helps the rules validate and reduces spam.

4) Testing steps
- Open your GitHub Pages site and the browser console.
- Try joining the chat with a nickname and send messages. Verify:
  - Messages appear in the chat and in the Firebase Console → Realtime Database → `messages`.
  - `presence` entries appear under `presence/<uid>` with `displayName`, `ts`, and `lastMsgTs`.
  - Rapidly sending multiple messages should trigger the client rate-limit (alert).
  - If you bypass client checks (e.g., manually send writes), server rules should reject writes that violate validation — check browser console for permission-denied errors.

5) Committing and pushing changes
```bash
git add docs/index.html docs/firebase-rules.json docs/FIREBASE_CHAT_SETUP.md
git commit -m "Add Firebase realtime chat integration, rules, and setup docs"
git push origin main
```

6) Notes & next steps
- Free Spark plan has usage limits; monitor reads/writes and set up moderation if public.
- For stricter rate-limits or atomic enforcement, implement Cloud Functions as a relay (verify and write messages server-side).
- Consider enabling Firebase Realtime Database backups and the Emulator for local testing.
