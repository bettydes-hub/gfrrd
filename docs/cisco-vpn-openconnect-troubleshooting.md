# Cisco VPN / OpenConnect troubleshooting — firewall.imagingak.com

**Date investigated:** 26 Oct 2026 (Africa/Nairobi)  
**Endpoint:** `https://firewall.imagingak.com` → `209.165.171.194:443`  
**Working client:** Cisco AnyConnect on phone (username + password only)  
**Failing client:** OpenConnect VPN for Windows `1.6.2-dirty`

## Verdict

OpenConnect fails **before** a successful XML authentication exchange because this Cisco ASA **rejects XML auth POSTs unless the HTTP `User-Agent` starts with `AnyConnect`**.

That matches a well-known Cisco / OpenConnect compatibility issue. Installing **Cisco Secure Client / AnyConnect** on the PC is expected to work (already proven on the phone with the same credentials). OpenConnect can also be made to work by forcing a compatible User-Agent — if the Windows GUI exposes that setting or you use the CLI.

TLS, DNS, and reachability are **not** the problem.

## What your logs already show (and what they do not)

From the 18:11–18:16 (local PC) snippet:

| Observation | Meaning |
|---|---|
| `POST https://firewall.imagingak.com/wimr1507` | Client is using the group URL alias (good — see below). |
| `Connected to 209.165.171.194:443` | DNS + TCP to the ASA work. |
| `non-CA certificate in the trusted list` (Microsoft Root Authority, etc.) | Benign Windows trust-store noise. Not a connection failure. |
| TLS 1.3 + AES-128-GCM | HTTPS negotiation succeeded. |
| Log ends at `XML` | The decisive HTTP status / XML body after that POST is missing. Capture the next lines. |

## Live endpoint findings (no credentials used)

### Certificate / identity

- Subject: `CN=*.imagingak.com`, Org: Imaging Associates (Anchorage, AK)
- Issuer: DigiCert Global G2 TLS RSA SHA256 2020 CA1
- Valid: 29 Jan 2026 → 1 Mar 2027
- SANs: `*.imagingak.com`, `*.connect49.com`

### Tunnel group behind `/wimr1507`

Web path `/wimr1507` maps to Cisco tunnel-group **`iapvpn`** (login redirect uses `tgroup=iapvpn`).

With a compatible client User-Agent and `group-access` set to `https://firewall.imagingak.com/wimr1507`, the ASA returns a normal username/password form:

```xml
<tunnel-group>iapvpn</tunnel-group>
...
<input type="text" name="username" .../>
<input type="password" name="password" .../>
```

That matches the phone flow (server + username + password, no extra group picker).

Connecting to the **root** URL without that group alias instead returns `<client-cert-request>` (certificate auth). Prefer the `/wimr1507` (or equivalent group) address on the PC as well.

### Smoking gun: User-Agent gate

XML auth `POST https://firewall.imagingak.com/` with Aggregate-Auth headers:

| User-Agent | HTTP result |
|---|---|
| `Open AnyConnect VPN Agent v9.12` (classic OpenConnect) | **404** empty body |
| `OpenConnect-GUI 1.6.2` | **404** empty body |
| `Cisco AnyConnect VPN Agent for Windows 4.10…` | **404** empty body |
| `Mozilla/5.0` | **404** empty body |
| `AnyConnect Windows 4.10.07061` | **200** XML auth-request |
| `AnyConnect-compatible OpenConnect VPN Agent v9.12` | **200** XML auth-request |

Rule observed on this server: User-Agent must start with the exact prefix **`AnyConnect`**.

When OpenConnect gets 404 on the XML POST, it typically falls back to obsolete non-XML auth, which then fails in confusing ways (“authentication did not complete”) even though the official Cisco client works.

## Recommended next steps (in order)

### Step 1 — Capture the rest of the OpenConnect log (no config changes)

Reproduce one failed connect and save lines **after** `XML`, especially:

- `Got HTTP response: HTTP/1.1 …`
- any `404 Not Found`
- `Failed to complete authentication` / form errors
- whether it falls back to `GET https://…`

If you see **404** on the XML POST, that confirms this report’s root cause on your PC.

### Step 2 — Prefer Cisco Secure Client on Windows (most reliable fix)

Because:

1. Phone already proves the ASA + your credentials work with Cisco AnyConnect.
2. This ASA clearly expects an official-style AnyConnect client identity.
3. IT may later enable hostscan/CSD, SAML, or client certs that OpenConnect handles poorly.

Install **Cisco Secure Client** (formerly AnyConnect) from your IT portal / approved package. Connect with:

- Server: `firewall.imagingak.com` **or** `firewall.imagingak.com/wimr1507` if IT gave that URL
- Username / password (same as phone)

No random registry or certificate store changes needed first.

### Step 3 — Optional: keep using OpenConnect with a forced User-Agent

Only if you intentionally want OpenConnect.

**CLI example:**

```bash
openconnect --protocol=anyconnect \
  --useragent="AnyConnect Windows 4.10.07061" \
  --user=YOUR_USERNAME \
  https://firewall.imagingak.com/wimr1507
```

Notes:

- Prefix must be `AnyConnect…` (not `Open AnyConnect…`).
- Keep the `/wimr1507` group path so you get username/password on `iapvpn`, not a client-cert challenge on the default group.
- OpenConnect-GUI 1.6.2 may not expose User-Agent in the UI; if it does not, use CLI or switch to Cisco Secure Client.

### Step 4 — Only if Cisco client also fails on the PC

Then compare PC vs phone systematically (still no random changes):

1. Confirm same server string and account.
2. Check Windows firewall / security suite intercepting TAP/Cisco adapters.
3. Ask IT whether Windows requires posture (HostScan/ISE), always-on VPN, or a machine certificate.
4. Collect Cisco Secure Client `AnyConnect.txt` / DART bundle for IT.

## What not to change yet

- Do not disable TLS verification / ignore certificate errors (cert is valid DigiCert).
- Do not delete Microsoft root certificates because of the “non-CA certificate” log lines.
- Do not force `--no-xmlpost` as a first fix (that is a legacy workaround and often worse on modern ASAs).
- Do not switch DNS or hosts-file entries; `firewall.imagingak.com` already resolves correctly to `209.165.171.194`.

## Summary

| Question | Answer |
|---|---|
| Why does OpenConnect fail? | Very likely ASA User-Agent filtering → XML POST **404** → broken auth fallback. |
| Are credentials wrong? | Unlikely — phone AnyConnect succeeds with the same account pattern. |
| Will Cisco Secure Client on the PC fix it? | **Yes, expected** — same protocol family as the working phone client. |
| Can OpenConnect work? | Likely yes **if** User-Agent is forced to start with `AnyConnect` and group URL `/wimr1507` is used. |

## Windows: which Cisco app to install

**Product name:** **Cisco Secure Client** (this is the current name; it still includes the **AnyConnect VPN** module). Older docs and IT emails may still say “AnyConnect”.

**Do not** download random “AnyConnect” `.exe` / `.msi` from third-party websites. Use one of the sources below.

### Where to get the installer (pick the first that works)

1. **Company IT / Imaging Associates** (best)  
   Ask for the Windows **Cisco Secure Client** (or AnyConnect) package they support for `firewall.imagingak.com`. Prefer their pre-approved build.

2. **VPN web portal after login**  
   In a browser, open `https://firewall.imagingak.com` (or `https://firewall.imagingak.com/wimr1507`), sign in with the same username/password as on your phone, then look for a **Download** / **AnyConnect** / **Secure Client** link.  
   Note: the portal does **not** expose a public installer without authentication.

3. **Cisco Software Download** (only if you or IT have a Cisco.com account with entitlement)  
   - Product: **Cisco Secure Client** (including AnyConnect), release **5.x**  
   - Windows file pattern: `cisco-secure-client-win-<version>-core-vpn-predeploy-k9.msi`  
     (or the larger `…-predeploy-k9.zip` / `…-predeploy-k9.msi` package)  
   - Official catalog: https://software.cisco.com/download/home/286330811  

You usually only need the **Core VPN** module. Skip Umbrella / AMP / ISE / NAM modules unless IT tells you to install them.

### Install steps on Windows 11

1. **Uninstall OpenConnect VPN for Windows** (Settings → Apps). Reboot if prompted. This avoids conflicting virtual adapters.
2. Run the Cisco `.msi` as Administrator. Accept defaults unless IT gave a custom package.
3. After install, start **Cisco Secure Client** (Start menu). Older installs may still show **Cisco AnyConnect Secure Mobility Client**.
4. In the VPN box, enter exactly one of:
   - `firewall.imagingak.com/wimr1507` (preferred — same group alias OpenConnect used)
   - or `firewall.imagingak.com` if IT says that is enough
5. Click **Connect**, enter the same **username** and **password** as on your phone.
6. Allow any Windows Firewall / driver prompts for the Cisco adapter.

### Success / failure checks

- **Success:** status becomes Connected; corporate resources load.
- **If it asks for a certificate** and your phone never did: you used the wrong group URL — retry with `/wimr1507`.
- **If credentials fail on PC but work on phone:** stop and ask IT (account may be device-restricted, or MFA/posture may apply only to Windows).
- **If install is blocked:** corporate PC policy may require IT to push the client via software center / Intune / SCCM.

### What not to do

- Do not keep OpenConnect and Cisco Secure Client both trying to own the VPN at the same time.
- Do not import custom root certificates for this server (DigiCert cert is already valid).
- Do not change DNS or hosts-file entries for `firewall.imagingak.com`.
