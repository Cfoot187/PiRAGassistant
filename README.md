#  Raspberry Pi RAG Email Assistant

This project sets up a **Retrieval-Augmented Generation (RAG) email assistant** on a **Raspberry Pi**. The assistant indexes work documentation and generates responses to emails using a **local LLM (TinyLlama / Phi-2)**.

##  Features
Runs a **lightweight LLM** locally on **Raspberry Pi**  
Uses **LlamaIndex + ChromaDB** to **search work documentation**  
**Drafts email replies** using an **efficient LLM**  
**(Optional)** Sends email responses automatically via **SMTP**  

---

##  Requirements
- Raspberry Pi (4GB RAM recommended)
- Python 3
- Llama.cpp (for running local LLMs)
- ChromaDB or FAISS (for document retrieval)
- SMTP access (if sending emails)

---

##  Installation
### 1 Install Dependencies
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3 python3-pip -y
pip3 install llama-index llama-index-llms-llama-cpp chromadb faiss-cpu pypdf
```

### 2️ Install Llama.cpp (for Local LLM Processing)
```bash
git clone https://github.com/ggerganov/llama.cpp.git
cd llama.cpp
make
```

### 3️ Download a Lightweight LLM
```bash
mkdir -p ~/models && cd ~/models
wget https://huggingface.co/TheBloke/TinyLlama-1B-Chat-GGUF/resolve/main/TinyLlama-1B-Chat.Q4_K_M.gguf
```

---

##  Index Work Documentation
Place your **work emails, PDFs, or documents** in `~/docs/` and run:
```python
from llama_index import VectorStoreIndex, SimpleDirectoryReader

# Load work documents
documents = SimpleDirectoryReader("~/docs").load_data()

# Create and save an index
index = VectorStoreIndex.from_documents(documents)
index.storage_context.persist(persist_dir="~/index")
```
This converts your **documents into a searchable vector index**.

---

##  Query Work Docs & Generate an Email Response
```python
from llama_index import StorageContext, load_index_from_storage
from llama_index.llms.llama_cpp import LlamaCPP

# Load Indexed Work Documents
storage_context = StorageContext.from_defaults(persist_dir="~/index")
index = load_index_from_storage(storage_context)

# Load TinyLlama Model
llm = LlamaCPP(model_path="~/models/TinyLlama-1B-Chat.Q4_K_M.gguf")

# Query Work Docs
email_question = "How do I respond to an email about compliance policies?"
retrieved_info = index.as_query_engine().query(email_question)

# Generate Email Reply
response = llm.complete("Write a professional email reply using this information: " + retrieved_info.response)
print("Suggested Email Reply:\n", response)
```

---

##  Send an Email Reply (Optional)
```python
import smtplib
from email.mime.text import MIMEText

msg = MIMEText(response)
msg["Subject"] = "Re: Compliance Policy Inquiry"
msg["From"] = "your-email@example.com"
msg["To"] = "recipient@example.com"

with smtplib.SMTP("smtp.gmail.com", 587) as server:
    server.starttls()
    server.login("your-email@example.com", "your-app-password")
    server.sendmail(msg["From"], [msg["To"]], msg.as_string())

print("Email Sent!")
```

_(For Gmail, you’ll need an **App Password** instead of your normal password.)_

---

## 🛠 Automate with a Cron Job
To **run this script automatically**, add it to a cron job:
```bash
crontab -e
```
Add this line to run it every 10 minutes:
```cron
*/10 * * * * python3 /home/pi/rag_email_assistant.py
```

---

##  Next Steps
- [ ] Add an interactive web dashboard for reviewing responses before sending.
- [ ] Expand document support (e.g., emails, TXT files, etc.).

**Contributions Welcome!** Feel free to fork and enhance the assistant.
