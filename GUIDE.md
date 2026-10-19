from pathlib import Path

content = r"""# GUIDE.md — Running Weather Forecast on Raspberry Pi 4

This guide documents the steps that resolved the startup and React/Vite errors for the `weather-forecast` project on a Raspberry Pi 4 running Raspberry Pi OS Bullseye.

## 1. Check the environment

Go to the project directory:

```bash
cd ~/Downloads/weather-forecast
```

Check the OS architecture and Node.js/npm versions:

```bash
uname -m
dpkg --print-architecture
node --version
npm --version
node -p "process.arch"
```

In the setup where this fix was tested, the results included:

- `uname -m`: `aarch64`
- `dpkg --print-architecture`: `armhf`
- `process.arch`: `arm`

The CPU supports 64-bit instructions, but the installed OS packages and Node.js are 32-bit ARM.

## 2. Back up the project configuration and dependency files

Before changing package versions, create backups:

```bash
cd ~/Downloads/weather-forecast

cp package.json package.json.backup
cp package-lock.json package-lock.json.backup
cp astro.config.mjs astro.config.mjs.backup
```

These backups are stored in the project directory. Keep them until the project has been tested successfully.

## 3. Install Astro versions compatible with this environment

The original Astro 7 dependency tree included Sätteri, whose installed version did not provide the required 32-bit ARM native binding (`@bruits/satteri-linux-arm-gnueabihf`).

Install the tested Astro 6 and integration versions:

```bash
npm install astro@6.3.5 @astrojs/netlify@7.0.13 @astrojs/react@5.0.7
```

Remove the separately installed Unified Markdown package that was tried during troubleshooting:

```bash
npm uninstall @astrojs/markdown-remark
```

Do not use `--force` or `--legacy-peer-deps` to bypass dependency errors. If npm reports a dependency or engine conflict, review that error before proceeding.

## 4. Set the Astro configuration

Open the configuration:

```bash
nano astro.config.mjs
```

Use this configuration, which preserves Tailwind, React, and the Netlify adapter without the Unified Markdown override:

```javascript
// @ts-check
import { defineConfig } from 'astro/config';
import tailwindcss from '@tailwindcss/vite';
import react from '@astrojs/react';
import netlify from '@astrojs/netlify';

export default defineConfig({
  vite: {
    plugins: [tailwindcss()],
    css: {
      transformer: 'lightningcss',
    },
    build: {
      cssMinify: 'lightningcss',
    },
  },
  output: 'server',
  devToolbar: { enabled: false },
  integrations: [react()],
  adapter: netlify(),
});
```

Save in Nano with `Ctrl+O`, press `Enter`, and exit with `Ctrl+X`.

## 5. Pin Vite to the major version required by Astro 6

The first run after downgrading Astro reported that Vite 8 was installed even though Astro 6 requires Vite 7. Add an npm override so the dependency tree uses Vite 7:

```bash
npm pkg set 'overrides.vite=^7'
npm install
```

Check the installed version:

```bash
npm ls vite
```

The resolved Vite version should be in the `7.x` series, not `8.x`.

## 6. Start the development server

Run:

```bash
npm run dev
```

Astro should report that the development server is ready. It typically provides URLs similar to:

```text
Local    http://localhost:4321/
Network  http://<raspberry-pi-ip>:4321/
```

Open the Network URL from another device on the same network. For the setup page, open:

```text
http://<raspberry-pi-ip>:4321/setup
```

Replace `<raspberry-pi-ip>` with the IP address shown by Astro (for example, `192.168.100.9`).

## 7. If you need to verify the installed dependencies

Run:

```bash
npm ls astro @astrojs/react @astrojs/netlify vite
```

This helps confirm that the installed Astro, React integration, Netlify adapter, and Vite versions match the working setup.

## Notes

- Do not delete `package-lock.json` or `node_modules` as a first response to the original error. In this case, the underlying issue was that the native Sätteri package required by the original Astro dependency tree was unavailable for the installed 32-bit ARM environment.
- Installing `@bruits/satteri-wasm32-wasi` directly with ordinary npm installation did not work on this platform because that package declares `wasm32` as its target CPU.
- The Astro/Vite downgrade and override are a tested workaround for this Raspberry Pi environment, not a general recommendation to use older versions for every deployment.
- Keep the Netlify adapter if the project is deployed to Netlify. A local Netlify/Deno warning is a separate issue from the original Sätteri native-binding error; if it reappears, investigate it separately rather than changing the Markdown or Vite configuration again.
- If restoring from backups becomes necessary, stop the dev server first. Restore only the files you intend to roll back; the backups may represent the pre-fix dependency state.

## Quick setup checklist

For a fresh copy of the project, after checking the environment and making backups:

1. Install the compatible Astro and integration versions from Step 3.
2. Remove `@astrojs/markdown-remark` as shown in Step 3.
3. Apply the `astro.config.mjs` configuration from Step 4.
4. Add the Vite override and run `npm install` from Step 5.
5. Verify with `npm ls vite`.
6. Start the project with `npm run dev`.
7. Open `/setup` using the Raspberry Pi's Network URL.
"""

path = Path("/mnt/data/GUIDE.md")
path.write_text(content, encoding="utf-8")
print(f"Created {path} ({path.stat().st_size} bytes)")
