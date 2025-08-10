# n8n_WhatsApp_assistant
This project allows you to control your Google Drive directly from WhatsApp using Twilio Sandbox and n8n.   You can list, delete, move, and (in progress) summarize files by sending commands from WhatsApp.

## 1) Prerequisites

- Docker & Docker Compose installed  
- ngrok account + authtoken

  
## 2) Step-by-step: Run n8n with Docker Compose

1) **Clone/Make project folder**  
   ```bash
   mkdir n8n-workflow && cd n8n-workflow
   ```
2) **Create `.env` & `docker-compose.yml`** using the samples above.  
3) **Start services**  
   ```bash
   docker compose up -d
   docker compose logs -f n8n
   ```
4) **Open n8n editor**  
   - Browser: `http://localhost:5678`  
   - First time pe onboarding flow aayega; complete it.

---

## 3) Expose Local n8n to Internet via ngrok

1) **Login & set authtoken** (one-time):
   ```bash
   ngrok config add-authtoken <YOUR_NGROK_AUTHTOKEN>
   ```
2) **Start tunnel**:
   ```bash
   ngrok http http://localhost:5678
   ```
3) **Copy the public URL** shown by ngrok (e.g., `https://abcd1234.ngrok.app`).  
4) **Update `.env` → `WEBHOOK_URL`** with that ngrok URL:
   ```ini
   WEBHOOK_URL=https://abcd1234.ngrok.app
   ```
5) **Restart n8n container** so it picks new URL:
   ```bash
   docker compose restart n8n
   ```

## 4) Using Webhook Nodes (Important Difference)

- **Production webhooks**:  
  `https://<ngrok-domain>/webhook/<your-path>`  
- **Editor/test webhooks** (Execute Node to test):  
  `https://<ngrok-domain>/webhook-test/<your-path>`

  
## 🚀 Features

### Implemented:
- `LIST /FolderName`
  - Lists all supported files inside a Google Drive folder.
- `DELETE /FolderName/file.ext`
  - Deletes a file from the given folder.
- `MOVE /SourceFolder/file.ext /TargetFolder`
  - Moves a file from one folder to another.
- Supports file types:
  - `.pdf` (Portable Document Format)
  - `.docx` (Microsoft Word)
  - `.txt` (Plain Text)
  - Google Docs (exported as text)

### Work in Progress:
- `SUMMARY /FolderName`
  - Downloads all files in the folder.
  - Extracts text.
  - Summarizes content using OpenAI GPT-4o.
  - Sends summarized text to WhatsApp.

---

## 🧠 Command Syntax

| Command | Example | Description |
|---------|---------|-------------|
| `LIST /ProjectX` | `LIST /Invoices2025` | Lists all supported files in the folder |
| `DELETE /ProjectX/file.pdf` | `DELETE /Invoices2025/report.pdf` | Deletes the file from ProjectX |
| `MOVE /ProjectX/file.pdf /Archive` | `MOVE /Invoices2025/report.pdf /2024Archive` | Moves file to Archive |
| `SUMMARY /ProjectX` | `SUMMARY /Invoices2025` | Summarizes content of all files |

---

## ⚙️ Setup Instructions

### 1️⃣ **Twilio Sandbox for WhatsApp**
1. Go to [Twilio Console WhatsApp Sandbox](https://www.twilio.com/console/sms/whatsapp/learn).
2. Click **"Activate Sandbox"**.
3. Save:
   - **Sandbox Number** (e.g. `+14155238886`)
   - **Join Code**
4. In n8n:
   - Create a **Webhook node**.
   - Copy the **Test URL**.
5. In Twilio Sandbox settings:
   - Paste the Test URL into **"When a message comes in"**.
6. On WhatsApp:
   - Send `join <your-join-code>` to the sandbox number.

---

### 2️⃣ **Google Drive OAuth2 Credentials**
1. Go to [Google Cloud Console](https://console.cloud.google.com/).
2. Create a **New Project**.
3. Enable:
   - **Google Drive API**
4. Go to:
   - **APIs & Services → OAuth consent screen**
   - Select **External**
   - Fill app details
5. Create OAuth2 credentials:
   - **Application type:** Web Application
   - Add redirect URI:  
     ```
     https://<your-n8n-domain>/rest/oauth2-credential/callback
     ```
6. Save **Client ID** and **Client Secret** in n8n's Google Drive credentials.
7. Authenticate.


---

## 📄 File Handling

- **LIST**:
  - Search files by folder name.
  - Filter only supported file types.
- **DELETE**:
  - Search for exact file.
  - Delete by ID.
- **MOVE**:
  - Get file ID from source.
  - Get destination folder ID.
  - Update file's parent folder.
- **SUMMARY** (In Progress):
  - Get all files in folder.
  - Filter `.pdf`, `.txt`, `.docx`, Google Docs.
  - Download/export files.
  - Extract text.
  - Send to GPT-4o for summarization.



---

## 🐛 Problems Faced

- **Webhook & Twilio**
  - Test sandbox only works for registered numbers.
- **Google Drive**
  - Google Docs require export before text extraction.
  - Deleted files still appeared in search (needed "trashed = false").
- **n8n Limitations**
  - Switch node sends to only one output → Used multiple Filter nodes.
  - Need "Always Output Data" enabled for empty search results.
- **File Handling**
  - PDF extraction fails for scanned docs without OCR.
  - Large files take longer to process.

---

## 📝 Next Steps

- Implement `SUMMARY`:
  - Handle text extraction for PDF/DOCX/TXT/Google Docs.
  - Summarize using GPT-4o.
  - Combine summaries and send back to WhatsApp.
- Add **logging** in Google Sheets.
- Add **confirmation** for `DELETE` and `MOVE` commands.
- #⚠️ Error Handling

- Folder not found →  
  `⚠️ Folder {folder} not found.`
- File not found →  
  `⚠️ File {file} not found in folder {folder}.`
- Empty folder →  
  `⚠️ No supported files found in {folder}.`
- Invalid command →  
  `❓ Invalid command. Available commands: LIST, DELETE, MOVE, SUMMARY`

---

## 📎 Credits

- **n8n** → Workflow automation
- **Twilio Sandbox** → WhatsApp entry point
- **Google Drive API** → File management
- **OpenAI GPT-4o** → Summarization

---
