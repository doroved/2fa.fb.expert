## 2fa.fb.expert — TOTP Code Generator

Minimal web app to generate 6‑digit TOTP (2FA) codes from a Base32 secret. Type or paste your secret to see the current code and the remaining time, and copy it in one click.

Русская версия: [README_RU.md](./README_RU.md)

### Features

- **TOTP generation**: compute a 6‑digit code from a Base32 secret
- **Timer**: display seconds left until the code refreshes (updates every second)
- **Copy to clipboard**: one‑click copy via “Copy Code”
- **Secret validation**: show an error if the secret is invalid
- **URL routing**: the secret is reflected in `/{SECRET}` so you can bookmark or share
- **Input UX**: auto uppercase and removal of non‑alphanumeric characters

### Tech Stack

- **Vue 3 + Vite**
- **TypeScript**
- **Tailwind CSS v4**
- **lucide-vue-next** (icons)
- **native-browser-otp** (`totp`, `timeLeft`) for code generation and countdown
- **vue-router**

### Getting Started

```sh
bun install
bun dev
```

### Build

```sh
bun run build
```

### Preview production build

```sh
bun preview
```

### How to Use

1. Paste your Base32 secret into the input (e.g., from a service’s 2FA settings).
2. If valid, the 6‑digit code and countdown timer will appear.
3. Click “Copy Code” to copy the current code to your clipboard.
4. The URL updates to `/{SECRET}` — bookmark or share for quick access.
5. If the secret is invalid, an “Invalid secret” message appears.

### Notes

- Input is converted to uppercase and stripped of non‑alphanumeric characters.
- On errors, the timer stops and the code resets to a safe state.
- Clipboard uses `navigator.clipboard` and may require HTTPS or permissions.

Created by [@doroved](https://t.me/doroved_stories)
