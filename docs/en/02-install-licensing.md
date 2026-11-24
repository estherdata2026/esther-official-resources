# 2. Installation & Licensing


### 2.1 Deploy in four steps (Windows preview build)

1. Download the zip package from [GitHub Releases](https://github.com/estherdata2026/esther-official-resources/releases) or [Gitee releases](https://gitee.com/esther2026/esther-official-resources/releases) and extract it anywhere on the target server (portable, no installer).
2. **Claim a trial license** (see [2.2](#22-licensing)): register on our website [shumai-data.com](https://shumai-data.com) and sign in to the console to claim an all-modules trial coupon (trial duration is shown on the coupon), then download the license file `esther.lic` from **License Management** and place it in the program directory.
3. Double-click `start_esther.bat` to start the server.
4. Open `http://<server-ip>:8000` in your browser and sign in with the administrator account.

> Starting without a license exits immediately with instructions to obtain a machine fingerprint and request a license — this is expected; complete the request flow in [2.2](#22-licensing) first.

**Changing the port**: set the `ESTHER_PORT` environment variable and restart.

### 2.2 Licensing

- **A valid license is required to start**: request a license file (`esther.lic`) before deployment; without one the server will not start.
- **Claim a trial license**: new users receive an all-modules trial coupon upon registration (trial duration is shown on the coupon):
  1. Register an account on [shumai-data.com](https://shumai-data.com) and sign in to the console — the trial coupon is claimed automatically;
  2. Run `esther-cli.exe license fingerprint` to get the machine fingerprint (64 hex characters), submit it and download the license file `esther.lic` from **License Management** in the console;
  3. Place `esther.lic` in the program directory and run `start_esther.bat`.
- **Checking status**: `esther-cli.exe license status` (mode / expiry / days left).
- **Expiry & migration**: once a license expires the server stops starting and resumes right after renewal; licenses are bound to the machine fingerprint — to migrate to another server, reset the binding yourself in **License Management** in the console (twice per calendar month).

### 2.3 Runtime data & accounts

- Runtime data is created automatically under `data/` (SQLite + Kuzu graph database) — no external database required.
- On first startup an administrator account **admin** (initial password **admin123**) is created automatically — keep it safe after signing in.

### 2.4 Security notes

- The program is a Nuitka native build; some antivirus products may false-positive on it — add it to the allowlist. Dependencies are statically linked; no VC++ runtime installation is required.
- `POST /api/analyze` currently has no authentication — deploy on an **internal network**. The `/api/v1/*` endpoints require a login token.

---
