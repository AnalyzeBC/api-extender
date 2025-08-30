# API-Extender

**API-Extender** (product) from _Vitae Springs Software LLC_ (DBA **API Extender**) is a native extension for Microsoft Dynamics 365 Business Central that exposes extended tables & custom fields through a clean, Power BI–ready API surface.

- **Download extension (.app):**  
  ➜ https://github.com/Vitae-Springs-Software-LLC/API-Extender/releases/latest/download/api-extender.app

- **Default configuration (CSV):**  
  ➜ https://github.com/Vitae-Springs-Software-LLC/API-Extender/releases/latest/download/defaults.csv

## Docs
- [BC API v2 vs. Extended Connector Q&A](docs/bc-api-v2-vs-extended-connector.md)

---

> Legal entity: **Vitae Springs Software LLC**  
> DBA: **API Extender**  
> Product name: **API-Extender**  
> Site: **https://apiextender.com** (redirects to **https://analyzebc.com**)

## ✨ What it does
- Select BC tables/fields that are safe and useful for reporting or integration.  
- Generate Query (API) objects with consistent naming, versions, and publisher metadata.  
- Optional **defaults.csv** to bootstrap a curated set of table/field selections.  
- Simple license activation call from Business Central.  
- Produces **app.json** and **appContents.al** from your selections for easy packaging.

## ✅ Requirements
- Microsoft Dynamics 365 Business Central (v22+ recommended)  
- Permission to install extensions in your tenant

## 🚀 Quick Start
**1) Install the extension**
- Download the `.app` from the Latest Release (or use the link in your welcome email).
- In Business Central (Sandbox recommended): **Tell Me** → **Extension Management** → **Upload Extension** → select the `.app`.

**2) Set up API-Extender**
- **Tell Me** → **API-Extender Setup**.  
- Enter your **Publisher** and the **Activation Code** from your welcome email.  
- (Optional) Click **Load Defaults** to populate selections from `defaults.csv`.

**3) Generate artifacts**
- **Generate appContents.al** → creates all Query (API) objects.  
- **Generate app.json** → creates a consistent `app.json` with dependencies and `idRanges`.

**4) Create a new Visual Studio Code project**
- Install VS Code: https://code.visualstudio.com/download  
- Install **AL Language** extension.  
- **Ctrl+Shift+P** → **AL: Go!** to scaffold a BC project.

**5) Build and publish your new app**
- Copy `appContents.al` and `app.json` into the project; remove the `Customer.al` example.  
- **Ctrl+Shift+P** → **AL: Download Symbols**  
- **F5** to **Publish (with Debugging)** to your Sandbox.  
- When ready, export the app from Extension Management, commit to source control (GitHub recommended), and deploy to Production per your process.

## 🔐 License Activation (web service details)
The extension posts this JSON payload to your gateway:
```json
{ "activationCode": "GUID-string", "publisher": "Your Publisher Name", "requestId": "Stable Tenant Hash" }
