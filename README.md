API Builder (Designed for Microsoft Dynamics Business Central)

Create lightweight, query-based APIs from selected Business Central tables and fields—no heavy web service projects, 
just point, click, and publish to extend your Business Central connector.

Latest downloads

Extension (.app):
https://github.com/Vitae-Springs-Software-LLC/APIBuilder/releases/latest/download/APIBuilder.app

Defaults (CSV):
https://github.com/Vitae-Springs-Software-LLC/APIBuilder/releases/latest/download/defaults.csv

Website / Help, Product page & docs: https://apibuilder.com


✨ What it does

Lets you select BC tables/fields that are safe and useful for reporting or integration.

Generates Query (API) objects with consistent naming, versions, and publisher metadata.

Ships with defaults.csv (optional) to bootstrap a curated set of table/field selections.

Utilizes a simple license activation call from Business Central.

Produces app.json and appContents.al from your selections for easy packaging.


✅ Requirements

- Microsoft Dynamics 365 Business Central (v22+ recommended).
- Permission to install extensions in your tenant.


🚀 Quick Start
1) Install the extension
    - Download APIBuilder.app from the Latest Release. If you downloaded from the welcome email, you don't need to download anything.

2) Open a Sandbox environment in Microsoft Business Central
    - Search: "Extension"
    - Upload/install APIBuilder.app (Extension Management).

3) Set up API Builder
    - Search "API Builder" (API Builder Setup)
    - Add the name that you want to publish the new app with in Publisher and the activation code that you received from the welcome email.
    - If you want a welcome email and a free trial, click here: (7-day free trial available, https://apibuilder.com)
    - Choose Load Defaults to populate the selections table.

4) Generate artifacts
    - Generate appcontents.al → creates appContents.al with all Query (API) objects.
    - Generate app.json → creates a consistent app.json with dependencies and idRanges.

5) Create new Visual Studio Code project
    - Download Visual Studio Code - https://code.visualstudio.com/download (first timers)
    - Open the Extensions icon in left menu bar (right click to show if necessary) and search for & load: AL Language extension for Microsoft Dynamics 365 Business Central
    = Command Pallette, AL:Go! (Ctrl+Shift+)

6) Build and publish your new app
    - Copy/paste app.contents.al and app.json from downloads directory to project directory. Delete Customer.al example file and overwrite app.json.
    - Optional - Export log from API Setup (right click on logo) to include logo in the app that you create and distribute to a production environment.
    - Command Pallette, AL: Download Symbols (Ctrl+Shift+)
    - Command Pallette, AL: Publish with Debugging (Ctrl+Shift+)
    - When you are ready, export the app from the Extensions Manager in Business Central, add version to source control (integrated GitHub recommended) and import to Production.


🔐 License Activation (web service details)

The extension posts this JSON payload to your gateway:

{
  "activationCode": "GUID-string",
  "publisher": "Your Publisher Name",
  "requestId": "Stable Tenant Hash"
}

200 → Activation successful (optionally returns activationDate)
201 → Activation exists but expired
Other statuses → An informative error is shown

The requestId is a stable, non-identifying hash derived from the tenant slug (for anonymous per-tenant telemetry/troubleshooting).


🧩 Files in this repo

APIBuilder.app and defaults.csv


📄 License

Copyright © Vitae Springs Software, LLC.
License: https://apibuilder.com/license/EULA.html
