# toastos-apps

App manifest and binaries for the toastOS App Store (`toastStore` in the toastOS GUI).

## How it works

`manifest.json` is a flat JSON array of app entries. Each entry's `sig` field
is an RSA-2048/SHA-256 signature over the raw `.tapp` ELF binary, verified
on-device against the public key baked into every toastOS kernel build
(`drivers/appsign.c`) before the app is ever written to disk. Unsigned or
tampered binaries are rejected by the client.

```json
{
  "name": "Hello World",
  "file": "HELLO.ELF",
  "url": "https://raw.githubusercontent.com/thetoasta/toastos-apps/main/HELLO.ELF",
  "version": "1.0",
  "developer": "thetoasta",
  "sig": "<hex RSA signature>"
}
```

## Publishing a new app

From the toastOS repo root:

```bash
export PATH=/usr/local/cross/bin:$PATH

# text app
x86_64-elf-gcc -m32 -ffreestanding -nostdlib -I apps/sdk -c apps/myapp.c -o myapp.o
x86_64-elf-gcc -m32 -ffreestanding -nostdlib -I apps/sdk -c apps/sdk/tapp_start.c -o tapp_start.o
x86_64-elf-ld -m elf_i386 -T apps/sdk/app_link.ld -o MYAPP.ELF myapp.o tapp_start.o

# sign it (private key kept offline, never in either repo)
python3 tools/sign_app.py /path/to/toastos_signing_private.pem MYAPP.ELF
```

Copy `MYAPP.ELF` into this repo, paste the printed `sig` hex into a new
`manifest.json` entry, commit, and push. The App Store polls this manifest
over HTTPS (`raw.githubusercontent.com/thetoasta/toastos-apps/main/manifest.json`).

## Apps in this repo

| App | File | Permissions | Notes |
|---|---|---|---|
| Hello World | `HELLO.ELF` | `PERM_PRINT` | SDK example / smoke test |
| toastNotes | `NOTES.ELF` | `PERM_GFX \| PERM_FS` | Visual notes app - first `PERM_GFX` app |
