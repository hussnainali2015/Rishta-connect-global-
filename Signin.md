# Setting up Google and Apple sign-in

Both need the site to be online at a real **https://** address (your domain). They do not work on an IP address or plain http.
Use the same address everywhere. Below it is written as `https://rishta.example.com`; replace it with yours.
In your settings (`.env`) `BASE_URL` must be exactly that address, with no slash at the end. After changing settings, restart the site.

## Google (free)
1. Open console.cloud.google.com, sign in, make a new project (top bar > New project).
2. Menu > APIs & Services > **OAuth consent screen** (now called "Google Auth Platform").
   - App name: Rishta Connect Global. Support email: yours.
   - Authorised domain: `example.com` (your domain without https or subdomain).
   - Privacy policy link: `https://rishta.example.com/privacy`. Terms link: `https://rishta.example.com/terms`.
   - Publishing status: press **Publish app** (In production). If it stays in "Testing", only people you add as Test users can sign in, and everyone else sees "Access blocked".
3. Menu > APIs & Services > **Credentials** > Create credentials > **OAuth client ID** > type **Web application**.
   - **Authorised JavaScript origins**: add `https://rishta.example.com` (nothing after .com: no slash, no path). Add `http://localhost:3000` too if you test on a computer.
   - Authorised redirect URIs: leave empty (the button does not use them).
4. Copy the **Client ID** (ends with `.apps.googleusercontent.com`) into `GOOGLE_CLIENT_ID`. Do not use the client secret; the site does not need it.
5. Restart the site. Google can take a few minutes (rarely up to an hour) to accept a new origin.

### Google errors and what they mean
| You see | Fix |
|---|---|
| "Error 400: origin_mismatch" or "The given origin is not allowed for the given client ID" | The address in the browser is not in Authorised JavaScript origins. Check https vs http, www vs no www, a trailing slash, typos. |
| "Access blocked: ... has not completed the Google verification process" or "Error 403: access_denied" | App is still in Testing. Publish it, or add your Google account under Test users. |
| "The OAuth client was not found" / "invalid_client" | `GOOGLE_CLIENT_ID` is wrong or from another project. |
| The site says "Google sign-in failed" | The server rejected the token. The server log says why; usually `GOOGLE_CLIENT_ID` on the server differs from the one the page uses, or the site was not restarted. |
| No Google button, message about the script | Something blocks `accounts.google.com` (ad blocker, firewall, no internet). |

## Apple (needs an Apple Developer Program membership, paid yearly)
1. developer.apple.com > Account > Certificates, Identifiers & Profiles > **Identifiers** > (+) **App IDs** > App. Name it, set a Bundle ID like `com.yourname.rishta`, tick **Sign in with Apple**, Register.
2. Identifiers > (+) **Services IDs**. Identifier: `com.yourname.rishta.web`. Register, then open it, tick **Sign in with Apple**, press **Configure**:
   - Primary App ID: the one from step 1.
   - Domains and Subdomains: `rishta.example.com` (no https, no slash).
   - Return URLs: `https://rishta.example.com/auth/apple/callback` (exactly this).
   - Next > Done > Continue > Save.
3. In your settings:
   - `APPLE_CLIENT_ID=com.yourname.rishta.web` (the **Services ID**, not the App ID)
   - `APPLE_REDIRECT_URI=https://rishta.example.com/auth/apple/callback` (identical to the Return URL)
4. Restart the site. The Apple button on the login window changes from "not configured" to "Continue with Apple".
No Apple key or secret is needed: the site only checks Apple's signed identity token.
If Apple asks you to verify the domain with a file, save the file Apple gives you as `public/.well-known/apple-developer-domain-association.txt`; the site serves it at `https://rishta.example.com/.well-known/apple-developer-domain-association.txt`.

### Apple errors
| You see | Fix |
|---|---|
| "invalid_client" | `APPLE_CLIENT_ID` is not the Services ID, or "Sign in with Apple" is not enabled/configured on it. |
| "Invalid web redirect url" / "invalid_request" | Return URL, domain, or `APPLE_REDIRECT_URI` differ. They must match exactly, with https. |
| Button says "Apple sign-in is not configured" | `APPLE_CLIENT_ID` is empty or the site was not restarted. |
| "Apple did not share a verified email" | The person previously used Sign in with Apple for this site. Apple > Settings > Apple ID > Sign in with Apple > stop using it for this site, then retry. |
| "Apple sign-in failed" on the site | See the server log; usually the Services ID does not match `APPLE_CLIENT_ID`. |

## Check your settings
`npm run check-env` now checks the Google and Apple lines.
When someone opens the login window, the site also shows a yellow note if it sees the usual problems (opened at the wrong address, not https, bad Google ID, scripts blocked).

## Still stuck?
Send me: the exact words of the error, the address in your browser bar, and which button (Google or Apple). Never send secret keys.
