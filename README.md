# Hydra Messenger: Decentralized Zero-Knowledge E2EE Node

Hydra is a lightweight, high-security asynchronous messenger node designed around the core principles of **Zero-Knowledge Architecture**, **End-to-End Encryption (E2EE)**, and **Ephemeral In-Memory Data Storage**. 

The platform guarantees that the hosting server possesses absolute zero knowledge regarding user identities, passwords, or communication content. All cryptographic operations occur strictly on the client side.

## 🛡️ Core Security Architecture

### 1. Zero-Knowledge Challenge-Response Authentication
Traditional password transmissions over networks are entirely eliminated. 
* **Key Generation:** During registration, a cryptographic asymmetric key pair is generated directly inside the user's browser. The private key is encrypted locally using the user's master password via **AES-GCM** and saved as a detached `.key` file.
* **Authentication:** Login is executed via a stateless **Challenge-Response protocol**. The server generates a unique random string (challenge), and the client signs it using their private key. The server verifies the signature against the stored public key. The server never sees, handles, or stores user passwords.

### 2. Client-Side End-to-End Encryption (E2EE)
* All message payloads are encrypted directly in the browser using **AES-GCM (128-bit)** before network transmission.
* Initialization Vectors (IV) are cryptographically generated on the fly for every single message packet.
* Cryptographic keys are initialized dynamically in the client’s volatile memory and are completely wiped upon session termination.

### 3. Volatile RAM-Only Storage & Multi-Layered TTL
To achieve an absolute anti-forensic footprint, data persistence on persistent drives is avoided for communication streams.
* **Server-Side Ephemeral Storage:** Real-time transport is handled via asynchronous WebSockets (`django-channels`). Active chat logs are stored strictly inside the volatile memory of a **Redis** instance.
* **Automated Expiry (TTL):** Chat history keys are bound to a strict **4-hour Time-To-Live (TTL)**. Once expired, Redis permanently flushes the data from RAM, leaving zero traces for forensic extraction.
* **Transit TTL:** Transient socket layers enforce a strict **10-second expiry** to automatically destroy stuck packets during network drops or ungraceful disconnections.

### 4. Client-Side Anti-Coercion Features (OpSec Overrides)
* **Dynamic Secret Kill Word:** Users can configure a custom panic phrase within their Privacy settings. If entering the secret folder under duress, submitting an incorrect or forced word triggers a silent **Master Wipe Sequence**.
* **Master Wipe Sequence:** Instantly clears all local storage, session storage, and drops client-side E2EE cryptographic keys, routing the interface back to a clean landing state.
* **Localized Backup Control:** History archiving is completely controlled by the user. Chat history can be imported or exported locally as plaintext `.txt` files compiled on the client machine, maintaining complete isolation from the hosting backend.

## 🛠️ Tech Stack

* **Backend Framework:** Django 5.x (Python 3.11+)
* **Asynchronous Networking:** Daphne ASGI Server & Django Channels 4.x
* **In-Memory Ledger:** Redis 7.x/8.x (RESP2 Communication Protocol)
* **Database (Profiles & Configurations Only):** PostgreSQL
* **Frontend Layer:** Vanilla JavaScript (Web Crypto API, WebSockets, Async/Await), CSS3 variables with adaptive themes (`Eclipse`, `Cyberpunk`, `Aurora`) and customizable blur properties.

## 🚀 Deployment & Local Node Testing

### 1. Prerequisites
Ensure you have Python 3.11+, PostgreSQL, and a running Redis instance on your machine.

### 2. Environmental Variables
Create a `.env` file in the root directory:
```text
SECRET_KEY=your_django_secret_key
DEBUG=True
ALLOWED_HOSTS=127.0.0.1, localhost
DB_NAME=hydra_db
DB_USER=postgres
DB_PASSWORD=your_secure_password
DB_HOST=127.0.0.1
DB_PORT=5432
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
```

### 3. Installation
```bash
# Clone the repository
git clone https://github.com
cd hydra-messager

# Set up virtual environment
python -m venv venv
source venv/Scripts/activate  # On Windows: venv\Scripts\activate

# Install required drivers and packages
pip install django daphne channels channels-redis redis python-decouple psycopg2 pillow
```

### 4. Initialize Node
```bash
python manage.py makemigrations
python manage.py migrate
python manage.py runserver 0.0.0.0:8000
```

## ⚖️ License & Disclaimer
This software is provided "as is", without warranty of any kind. Developed exclusively for secure, decentralized sandboxed environments and advanced cryptographic research.
