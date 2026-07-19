## 🤖 AI-Agent Setup Instructions

> **Copy the prompt below and hand it to any AI agent (Claude, GPT, Gemini, Cursor, etc.) running on the Snapdragon laptop. It will set up Gyaan end-to-end.**

### ⚙️ Agent Prompt (click to copy)

````markdown
You are setting up the **Gyaan** Next.js app on a **Snapdragon X Elite laptop (Windows ARM64, 32GB RAM)**. The laptop is fresh — assume nothing dev-related is installed. A zip file `gyaan-dragon.zip` has been copied to the Desktop.

Execute these steps IN ORDER. Stop and report if any step fails. Do not skip verification.

**PREREQUISITES**
- Target: Snapdragon X Elite, Windows 11 ARM64
- The zip lives at `C:\Users\<user>\Desktop\gyaan-dragon.zip`
- You have admin PowerShell access

---

### STEP 1 — Install Node.js LTS (ARM64)
```powershell
winget install OpenJS.NodeJS.LTS
# Verify:
node --version    # expect v20.x or v22.x
npm --version
```
If winget fails, download the arm64 MSI from https://nodejs.org/dist/latest-v22.x/

### STEP 2 — Unzip the source
```powershell
Expand-Archive "C:\Users\<user>\Desktop\gyaan-dragon.zip" "C:\Users\<user>\gyaan"
cd C:\Users\<user>\gyaan
```

### STEP 3 — Configure environment
```powershell
Copy-Item .env.example .env.local
```
Then edit `.env.local` to contain EXACTLY these values (overwrite the LLM section):
```
OPENAI_API_KEY=geniex
OPENAI_BASE_URL=http://127.0.0.1:18181/v1
DEFAULT_MODEL=openai:ai-hub-models/Qwen3-4B-Instruct-2507
ALLOW_LOCAL_NETWORKS=true
GYAAN_GRID_DSL=true
LLM_THINKING_DISABLED=true
LLM_STRIP_THINK_BLOCKS=true
```

### STEP 4 — Install dependencies
```powershell
npm install
```
**IMPORTANT:** Use `npm`, NOT `pnpm` — pnpm has a known blank-terminal bug on Windows ARM64.
The postinstall script builds 5 workspace subpackages (mathml2omml, pptxgenjs, @gyaan/dsl, importer, renderer). Expect 3–5 minutes. If one subpackage fails, report which.

### STEP 5 — Install GenieX (Qualcomm NPU server)
```powershell
# Option A: pip
pip install -i https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple geniex

# Option B: installer from https://github.com/qualcomm/GenieX/releases
```
Verify: `geniex --version`

### STEP 6 — Pull the NPU model (one-time, ~3GB download)
```powershell
geniex pull ai-hub-models/Qwen3-4B-Instruct-2507
```

### STEP 7 — Open firewall for phone access (admin)
```powershell
New-NetFirewallRule -DisplayName "Gyaan 3000" -Direction Inbound -Protocol TCP -LocalPort 3000 -Action Allow -Profile Private,Public
```

### STEP 8 — Find the laptop's LAN IP
```powershell
ipconfig
```
Note the IPv4 Address (e.g., `10.249.126.74`). The phone will use this.

### STEP 9 — Start GenieX server (Terminal 1 — keep running)
```powershell
geniex serve --host 0.0.0.0 --port 18181
```
Verify in another terminal:
```powershell
curl http://127.0.0.1:18181/v1/models
```
Expect JSON listing `ai-hub-models/Qwen3-4B-Instruct-2507`.

### STEP 10 — Start Gyaan (Terminal 2 — keep running)
```powershell
cd C:\Users\<user>\gyaan
npm run dev -- --hostname 0.0.0.0
```
If Turbopack errors (`Cannot find module ... win32-arm64`), set `$env:TURBOPACK=0` and retry.

### STEP 11 — Verify
- On the laptop: open `http://localhost:3000` → Gyaan loads.
- On a phone (same Wi-Fi): open `http://<laptop-ip>:3000` → Gyaan loads.
- Generate a 5-slide classroom on any topic → confirm slides render + speech plays.

### STEP 12 — Set Wi-Fi network to Private (if phone can't connect)
`Settings → Network & Internet → Wi-Fi → <network> → Network profile type → Private`

### SUCCESS CRITERIA
- [ ] `node --version` ≥ v20
- [ ] `npm install` completed with 0 errors
- [ ] `geniex serve` running and `curl` returns the model
- [ ] `npm run dev` shows "Ready" on http://0.0.0.0:3000
- [ ] Laptop browser loads Gyaan
- [ ] Phone browser loads Gyaan over LAN
- [ ] A classroom generates and plays back

### IF SOMETHING FAILS — report:
- Which step, exact error text, `node --version`, OS build (`winver`).
- Do NOT improvise fixes — stop and ask.
````

---

### 📋 Quick Command Reference (all-in-one, for humans)

```powershell
# One-time setup
winget install OpenJS.NodeJS.LTS
Expand-Archive gyaan-dragon.zip C:\Users\you\gyaan
cd C:\Users\you\gyaan
Copy-Item .env.example .env.local   # then edit the 4 LLM values
npm install
pip install -i https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple geniex
geniex pull ai-hub-models/Qwen3-4B-Instruct-2507
New-NetFirewallRule -DisplayName "Gyaan 3000" -Direction Inbound -Protocol TCP -LocalPort 3000 -Action Allow -Profile Private,Public

# Every run (2 terminals)
# Terminal 1:
geniex serve --host 0.0.0.0 --port 18181
# Terminal 2:
npm run dev -- --hostname 0.0.0.0
```

### 🌐 Access
- Laptop: `http://localhost:3000`
- Phone (same Wi-Fi): `http://<laptop-ip>:3000`
