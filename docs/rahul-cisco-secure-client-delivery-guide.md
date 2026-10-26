# Guide for Rahul: Get and send the Windows Cisco VPN client

**Purpose:** Provide the Windows VPN installer so a PC user can connect to the company VPN the same way the phone already does with Cisco AnyConnect.

**VPN server:** `firewall.imagingak.com`  
**Preferred connection URL:** `firewall.imagingak.com/wimr1507`  
**Client to send:** Cisco Secure Client (includes AnyConnect VPN) for Windows

---

## What you need to send

Send **one** Windows installer package for **Cisco Secure Client / AnyConnect VPN**.

| Prefer | Acceptable older name |
|---|---|
| `cisco-secure-client-win-<version>-core-vpn-predeploy-k9.msi` | `anyconnect-win-<version>-core-vpn-predeploy-k9.msi` |

Also acceptable:
- A company zip that **contains** one of those MSI files
- An IT-approved full Secure Client predeploy package, if that is what Imaging Associates normally uses

**Do not send:**
- Random “AnyConnect” downloads from third-party websites
- OpenConnect
- Extra modules (Umbrella, AMP, ISE, NAM) unless your standard company package includes them on purpose

---

## Step 1 — Prefer an existing company copy

1. Check whether Imaging Associates already has an approved Windows VPN client package in:
   - software share / software center
   - Intune / SCCM / endpoint management
   - previous IT VPN deployment files
2. If you find an approved **Cisco Secure Client** or **AnyConnect** Windows installer, skip to **Step 3** (how to send it).
3. If you do not have one, continue to **Step 2**.

---

## Step 2 — Download from Cisco (if no local package)

1. Sign in to Cisco Software Download:  
   https://software.cisco.com  
   Use a Cisco.com account tied to the company contract/license.
2. Search for: **Cisco Secure Client** (including AnyConnect).
3. Open the product downloads for **Secure Client 5.x** (current line).
4. Choose the **Windows** package.
5. Download one of these:
   - **Best:** Core VPN predeploy MSI  
     Example pattern:  
     `cisco-secure-client-win-5.x.x.xxxx-core-vpn-predeploy-k9.msi`
   - **Or:** full Windows predeploy zip/MSI, then use the Core VPN installer inside it if separate modules are listed
6. Save the file somewhere you can share securely.

**If Cisco.com blocks the download:** the account may lack entitlement. Use an account that owns the AnyConnect/Secure Client license, or use your existing internal package from Step 1.

**Optional alternate source:**  
If the VPN web portal is configured to offer client download:

1. Open https://firewall.imagingak.com in a browser  
2. Sign in with an account that can reach the download page  
3. Look for **Download AnyConnect** / **Cisco Secure Client** for Windows  
4. Download that package and continue to Step 3

---

## Step 3 — Send the file securely

Send the installer to the PC user using a normal company channel:

1. Email attachment (if file size is allowed), **or**
2. OneDrive / SharePoint / Teams link, **or**
3. IT ticket / software request attachment

In the message, include these notes for the user:

```text
Windows Cisco VPN installer attached/linked.

Please:
1) Uninstall "OpenConnect VPN for Windows" first
2) Install this MSI (Cisco Secure Client / AnyConnect)
3) Open Cisco Secure Client
4) Connect to: firewall.imagingak.com/wimr1507
5) Use the same VPN username and password as on your phone

You only need the VPN/Core VPN client. Contact me if Windows blocks the install.
```

---

## Step 4 — Confirm they install the right thing

After they receive the file, they should:

1. Uninstall **OpenConnect VPN for Windows**
2. Run the `.msi` you sent (Administrator if prompted)
3. Open **Cisco Secure Client** (may still appear as **Cisco AnyConnect Secure Mobility Client**)
4. Enter: `firewall.imagingak.com/wimr1507`
5. Sign in with VPN username + password

### Expected result
- Client connects successfully
- Same credentials that already work on the phone should work on the PC

### If it fails
| Symptom | What to check |
|---|---|
| Install blocked | Company policy / SmartScreen / need IT push via Intune-SCCM |
| Asks for a certificate (phone never did) | Confirm they used `/wimr1507`, not only the bare hostname |
| Login rejected on PC but works on phone | Account restriction, MFA, or Windows posture policy — review on the ASA/IT side |
| Wrong app installed | Confirm file was Cisco Secure Client / AnyConnect Core VPN, not OpenConnect |

---

## Short checklist for Rahul

- [ ] Locate or download Windows **Cisco Secure Client / AnyConnect Core VPN** MSI  
- [ ] Confirm filename looks like `cisco-secure-client-win-…-predeploy-k9.msi` (or AnyConnect equivalent)  
- [ ] Do **not** use a random internet mirror  
- [ ] Send file via company email/SharePoint/Teams  
- [ ] Tell user to uninstall OpenConnect first  
- [ ] Tell user to connect to `firewall.imagingak.com/wimr1507`  

That is all that is required on your side for the PC VPN client delivery.
