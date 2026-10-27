# Deploying Yahoo Finance Ad Blocker with Intune

The extension is hosted on GitHub Pages, and Edge downloads it from there:

- Update file: https://zarbam.github.io/yahoo-finance-adblock/update.xml
- Package: https://zarbam.github.io/yahoo-finance-adblock/yahoo-finance-adblock-2.0.0.crx
- Extension ID: `gcofienjfpjmdlofdldanpflgopbmbig`

Requirements:

- Windows PCs enrolled in Intune.
- Umbrella must allow `zarbam.github.io`.

## 1. Create the policy

1. Open the Intune admin center: https://intune.microsoft.com
2. Go to **Devices → Configuration → Create → New policy**.
3. Set **Platform** to *Windows 10 and later* and **Profile type** to *Settings catalog*. Click **Create**.
4. Name it `Edge - Yahoo Finance Ad Blocker`, then click **Next**.
5. Click **Add settings** and search for `Control which extensions are installed silently`.
   Pick the one under **Microsoft Edge\Extensions**, not the one marked "(User)".
6. Set it to **Enabled** and add this value:

   ```
   gcofienjfpjmdlofdldanpflgopbmbig;https://zarbam.github.io/yahoo-finance-adblock/update.xml
   ```

7. Optional: pin the icon to everyone's toolbar.
   1. Add the setting **Configure extension management settings** from the same Microsoft Edge\Extensions category.
   2. Set it to **Enabled** and paste:

   ```json
   {"gcofienjfpjmdlofdldanpflgopbmbig": {"toolbar_state": "force_shown"}}
   ```

8. Click **Next** past Scope tags.
9. Under **Assignments**, assign the policy to a test group containing only your device first.
10. Click **Create**.

## 2. Test on one PC

1. Pull the policy right away: **Settings → Accounts → Access work or school → your account → Info → Sync**.
2. Fully close Edge and reopen it.
3. Open `edge://policy` and check:
   - `ExtensionInstallForcelist` is listed;
   - its status is **OK**.
4. Open `edge://extensions` and check that Yahoo Finance Ad Blocker is there and marked as managed by your organization.
5. Open https://finance.yahoo.com and check:
   - the icon shows a count;
   - clicking it lists the blocked companies;
   - the page and the video work.

## 3. Roll out

Edit the policy's **Assignments** and add the user or device groups that should get it.
Each PC installs the extension the next time it syncs with Intune and Edge restarts.

## Updating the extension

Intune doesn't need to change. Edge checks `update.xml` every few hours and installs new versions by itself.

1. Raise `version` in `manifest.json`.
2. Rebuild and sign the package:
   ```
   python3 build.py https://zarbam.github.io/yahoo-finance-adblock/
   ```
   This needs `../yahoo-finance-adblock-signing-key.pem`. If you lose that key, the next build gets a different extension ID and every PC treats it as a new extension. Back it up and never upload it.
3. Publish the new files:
   ```
   cd dist && git add update.xml *.crx && git commit -m "vX.Y.Z" && git push
   ```
   The `gh` CLI must be signed in as `zarbam`. Check with `gh auth status`, and switch with `gh auth switch`.

## Removing it

Delete the policy, or remove its assignment. Edge uninstalls the extension on the next sync.

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| Policy missing from `edge://policy` | The device hasn't synced yet. Sync it, or check its status in Intune. |
| Policy shows an error, extension not installed | The device isn't Intune-enrolled, or the value has a typo. It must be exactly `ID;URL`. |
| Policy OK, extension not installed | Umbrella or a proxy is blocking `zarbam.github.io`. Open the update.xml link in Edge on that PC to check. |
| "This extension is blocked by your administrator" | Another policy is blocking all extensions. Add `gcofienjfpjmdlofdldanpflgopbmbig` to **Allow specific extensions to be installed**. |
