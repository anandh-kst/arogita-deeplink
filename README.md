# Innosify Mobile Deep Links

This project is a static Vercel host for Android App Links and iOS Universal Links at `link.innosify.com`. It does not contain a website, UI, application server, or runtime dependencies.

## Replace the placeholders

Before deploying, replace these values:

- `YOUR_ANDROID_PACKAGE_NAME` in `public/.well-known/assetlinks.json` with the Android application ID/package name.
- `YOUR_SHA256_CERTIFICATE_FINGERPRINT` in `public/.well-known/assetlinks.json` with the SHA-256 fingerprint for the certificate that signs the installed Android app.
- `YOUR_TEAM_ID` in `public/.well-known/apple-app-site-association` with the Apple Developer Team ID.
- `YOUR_BUNDLE_ID` in `public/.well-known/apple-app-site-association` with the iOS app bundle identifier.

For iOS, the combined value must be `YOUR_TEAM_ID.YOUR_BUNDLE_ID`, for example `ABCDE12345.com.example.app` after replacement. Do not include spaces, and do not leave the placeholder values in production.

## Deploy with Vercel CLI

1. Install the Vercel CLI if needed:

   ```bash
   npm install --global vercel
   ```

2. From this project directory, authenticate and deploy:

   ```bash
   vercel login
   vercel
   ```

3. Follow the prompts to create or select the Vercel project. When ready to deploy to production, run:

   ```bash
   vercel --prod
   ```

This is a static deployment. No build command, framework, server, or dependencies are required.

## Connect `link.innosify.com`

1. Open the Vercel project dashboard.
2. Go to **Settings > Domains**.
3. Add `link.innosify.com`.
4. In the DNS provider for `innosify.com`, create the DNS record Vercel displays. For a subdomain, this is normally:

   - Type: `CNAME`
   - Name/Host: `link`
   - Target/Value: `cname.vercel-dns.com`

   Use the exact target shown by Vercel if it differs. Remove conflicting records for the same host. Vercel will issue HTTPS after the domain is correctly pointed and verified.

No website is required. The subdomain is used only to host the two mobile association files. Vercel is being used only as a static host for those files.

## Required verification URLs

After DNS and HTTPS are active, these exact URLs must return HTTP 200 and the JSON shown in the files:

- https://link.innosify.com/.well-known/assetlinks.json
- https://link.innosify.com/.well-known/apple-app-site-association

Both responses must have `Content-Type: application/json`, must not redirect, and must not require authentication.

You can inspect the headers with:

```bash
curl -I https://link.innosify.com/.well-known/assetlinks.json
curl -I https://link.innosify.com/.well-known/apple-app-site-association
```

## Find the Android SHA-256 fingerprint

Use the fingerprint for the signing certificate of the build being tested or released.

For a local keystore, run:

```bash
keytool -list -v -keystore path/to/keystore.jks -alias YOUR_KEY_ALIAS
```

Copy the `SHA256` value and format it as uppercase hexadecimal pairs separated by colons. For Google Play releases, check the Play Console under **Release > Setup > App integrity > App signing key certificate**. The Play App Signing certificate fingerprint is commonly the required value for apps installed from Google Play; a locally sideloaded build may use a different debug or upload certificate.

## Find the iOS Team ID and Bundle ID

- **Team ID:** In Apple Developer, open **Membership details** and copy the **Team ID**. It can also be found in the Apple Developer account or signing configuration.
- **Bundle ID:** In Xcode, select the app target, open **Signing & Capabilities**, and read **Bundle Identifier**. It must match the registered App ID exactly.

Put the two values together in `appIDs` as `TEAM_ID.BUNDLE_ID`.

## Testing

### Android App Links

1. Replace the Android placeholders and deploy the files.
2. Confirm the verification URL is publicly reachable with HTTP 200 and no redirect.
3. Build and install the Android app signed with the certificate whose fingerprint is in `assetlinks.json`.
4. Open an HTTPS link on `link.innosify.com`, such as `https://link.innosify.com/test`.
5. Confirm Android opens the app directly. On a development device, Android app-link status can also be inspected in the app's **Open by default** settings or with Android Debug Bridge tools appropriate to the OS version.

### iOS Universal Links

1. Replace the Team ID and Bundle ID placeholders and deploy the file.
2. Confirm the AASA URL is publicly reachable with HTTP 200 and `Content-Type: application/json`.
3. Enable the **Associated Domains** capability in the app and add `applinks:link.innosify.com`.
4. Build and install the app on a physical iOS device.
5. Tap an HTTPS link from Notes, Messages, Mail, or another supported app. iOS should open the installed app rather than Safari. Universal Links generally do not trigger when a URL is typed directly into Safari.

## Common mistakes

- **Incorrect Android SHA-256 fingerprint:** The fingerprint must match the certificate signing the installed APK. Debug, upload, Play App Signing, and old release certificates can all differ.
- **Incorrect iOS Team ID or Bundle ID:** Both values must belong to the same Apple app, and the combined `TEAM_ID.BUNDLE_ID` must be exact.
- **Redirects:** Do not redirect either `.well-known` URL, including HTTP-to-HTTPS or domain redirects at the association-file path. The final HTTPS URLs must respond directly.
- **Incorrect content types:** Both files must be served as `application/json`. The iOS file must be named exactly `apple-app-site-association` with no `.json` extension.
- **Wrong path or casing:** The directory must be `.well-known`, and the filenames are case-sensitive.
- **DNS or TLS not ready:** Verify that `link.innosify.com` resolves to the Vercel project and has a valid HTTPS certificate before testing on a device.
- **Stale app association data:** Android and iOS may cache association results. Reinstall the app or wait for the platform to refresh its verification data after correcting the files.
