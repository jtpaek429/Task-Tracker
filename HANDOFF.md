# Handoff — External Readiness Pass

## What was built

### 1. Dev mode restriction
- `🧪 Dev` button is hidden by default via `style="display:none"` on the element
- `onSignedIn()` shows it only for `jpaek429@gmail.com` and `test@test.com` (array `DEV_ALLOWED_EMAILS`)
- `onSignedOut()` hides it again

### 2. User area → dropdown
- Replaced flat email + buttons with a single email button that opens a dropdown
- Dropdown: Sign out / Delete account…
- Closes on Esc or click outside
- `toggleUserMenu()` / `closeUserMenu()` in JS

### 3. Account deletion
- **UI**: dropdown → confirmation modal with warning → red "Delete my account" button
- **Edge Function**: `supabase/functions/delete-account/index.ts`
  - JWT verification disabled on the function (we verify inside via `getUser()`)
  - Deletes tasks first (no `ON DELETE CASCADE` on the FK to `auth.users`)
  - Then calls `auth.admin.deleteUser(uid)` using `SERVICE_ROLE_KEY` secret
  - Note: Supabase doesn't allow `SUPABASE_` prefix for secret names → use `SERVICE_ROLE_KEY`
- **After deletion**: modal closes → sign out → "Your account has been deleted." success banner on auth overlay
- Button state resets each time the modal opens (prevents stuck "Deleting…" state across sessions)

### 4. Password reset flow
- "Forgot password?" link below password field (sign-in tab only)
- `showAuthView('forgot')` → email input → `resetPasswordForEmail(email, { redirectTo: window.location.origin })`
- On link click: Supabase redirects back with a PKCE code → `getSession()` exchanges it → `onAuthStateChange` fires with `PASSWORD_RECOVERY`
- `_inRecoveryFlow` flag blocks the subsequent `SIGNED_IN` event from signing the user in
- Reset form: new password input → `updateUser({ password })` → sign out → back to sign-in with success message

### 5. Email confirmation fix
- Root cause 1: Supabase Site URL was `https://mytodo.page/**` (wildcard in wrong place) — fixed to `https://mytodo.page`
- Root cause 2: Preview URLs weren't in allowed redirect URLs — added `https://*.vercel.app/**`
- Root cause 3: `flowType: 'implicit'` was set on the client but Supabase's server always sends PKCE codes (`?code=xxx`) in email links regardless — removed, back to default PKCE
- `onAuthStateChange` must be registered before `getSession()` so `PASSWORD_RECOVERY` is caught during the PKCE code exchange

### 6. Resend SMTP
- Configured in Supabase → Project Settings → Auth → SMTP
- Host: `smtp.resend.com`, port `465`, username `resend`, password = Resend API key
- Sender: `noreply@mytodo.page` (domain verified in Resend with DNS records in Vercel)
- Removes Supabase's built-in 2 emails/hour rate limit

### 7. Friendly auth error messages
- `friendlyAuthError(error)` helper detects rate-limit errors (status 429, message contains "rate limit") and returns human-readable text
- Applied to `submitAuth()` (sign-in + sign-up) and `sendPasswordReset()`

## Supabase dashboard config checklist
- [ ] Site URL: `https://mytodo.page`
- [ ] Redirect URLs: `https://mytodo.page/**`, `https://*.vercel.app/**`, `http://localhost:*/**`
- [ ] SMTP: Resend credentials (see above)
- [ ] Edge Function `delete-account`: deployed, JWT verification OFF
- [ ] Secret `SERVICE_ROLE_KEY`: set to Supabase service role key

## Key gotchas for future sessions
- **`SUPABASE_` prefix banned** for Edge Function secrets — use `SERVICE_ROLE_KEY` not `SUPABASE_SERVICE_ROLE_KEY`
- **No `ON DELETE CASCADE`** on `tasks.user_id` → must delete tasks before deleting auth user
- **Edge Function JWT verification** must be disabled — we verify the JWT inside the function ourselves
- **`onAuthStateChange` before `getSession()`** — order matters; reversing it causes `PASSWORD_RECOVERY` to be missed
- **`_inRecoveryFlow` flag** — Supabase fires `SIGNED_IN` right after `PASSWORD_RECOVERY`; without this flag the user gets signed in before they can set a new password

## What's still open
- DEV_AUTO_LOGIN (`test@test.com`) gives a 400 on preview URLs — credentials may be stale or account deleted
- Account deletion confirmed working; password reset confirmed working; email confirmation confirmed working
- Privacy policy may need updating to mention account deletion is now supported
