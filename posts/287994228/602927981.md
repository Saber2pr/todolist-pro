# FAQ & Troubleshooting

##  macOS Installation & Launch Issues

### Issue: "App is damaged and can't be opened. You should move it to the Trash."
**Cause:** This is a standard macOS **Gatekeeper** security warning. It occurs when an app is downloaded outside the App Store and has not been officially signed/notarized with an Apple Developer certificate.

**Solutions:**

#### Solution 1: Open via Finder Context Menu (Easiest)
1. Open **Finder** and locate the app (`.app` file).
2. Press and hold the **`Control` key**, then **right-click** the app icon.
3. Select **"Open"** from the context menu.
4. Click **"Open"** again in the confirmation dialog. *(You only need to do this once)*

#### Solution 2: Clear the Quarantine Flag via Terminal
If the right-click method fails, macOS has locked the file. You can manually clear the restriction:
1. Open the **Terminal** app.
2. Type the following command (make sure to leave a **space** at the end):
```bash
sudo xattr -r -d com.apple.quarantine 
```

3. **Drag and drop** the app icon from Finder directly into the Terminal window to auto-complete the path, then press **Enter**.
4. Type your Mac login password (characters won't show) and press **Enter**.
5. Launch the app normally.
