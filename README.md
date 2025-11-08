🧠 Secure FastAPI + PostgreSQL Data Entry System

A Jupyter-integrated backend with AES encryption, JWT authentication, and GUI-based user interaction.

📘 Table of Contents

Overview

System Architecture

Technology Stack

Project Directory Structure

Environment Configuration

Database Schema

Users

AES Keys

Key Logs

Entries

Orders

Order Logs

Logs

Entity Relationship Model

FastAPI Application Overview

Graphical User Interface (GUI)

Security Model

Deployment Notes

🧩 Overview

This system is designed as a secure, full-stack FastAPI service capable of receiving and storing structured data through a Tkinter-based desktop GUI, with data persisted in a PostgreSQL 18 database.

It integrates AES encryption for sensitive fields, JWT-based session authentication, and an optional key rotation mechanism for cryptographic hygiene.

This project bridges traditional industrial data entry environments (via GUI) and modern backend practices (FastAPI, PostgreSQL, and cryptography), suitable for research, production, and automation contexts.

⚙️ System Architecture
┌────────────────────────┐
│       GUI Client       │
│ (Tkinter + Requests)   │
│ - User Login/Register  │
│ - Data Entry (Orders)  │
│ - Logs Display         │
└────────────┬───────────┘
             │ REST API (HTTPS, JWT)
             ▼
┌────────────────────────┐
│       FastAPI Server   │
│ - Auth (JWT)           │
│ - AES Encryption       │
│ - CRUD for Users,      │
│   Orders, Logs, Keys   │
│ - Audit Middleware     │
└────────────┬───────────┘
             │ psycopg2 / SQLAlchemy
             ▼
┌────────────────────────┐
│     PostgreSQL 18      │
│ - Secure Schema        │
│ - Key/Log Management   │
│ - Triggers & Indices   │
└────────────────────────┘

🧰 Technology Stack
Layer	Component	Description
Backend	FastAPI	High-performance Python web framework
Database	PostgreSQL 18	Relational backend for persistent storage
Frontend (Local)	Tkinter GUI	Simple desktop interface for CRUD operations
ORM / Query	psycopg2 / SQLAlchemy	Secure parameterized queries
Auth	JWT (PyJWT / jose)	JSON Web Tokens for authentication
Encryption	AES (Fernet / Cryptography)	Symmetric encryption for data fields
Secrets	.env + dotenv	Environment-based secrets management
Runtime	uvicorn	ASGI server for FastAPI
Integration	Jupyter Notebook	Optional build and deployment automation
🗂️ Project Directory Structure
FastAPI_Server/
│
├── .env
├── main.py                 # FastAPI entry point
├── server.py               # Router + API definitions
├── database.py             # Connection and schema logic
├── models.py               # SQLAlchemy models
├── gui.py                  # Tkinter client application
├── schema.sql              # PostgreSQL schema
├── requirements.txt
└── README.md               # Documentation (this file)



🧱 Database Schema

All objects are created under the public schema.

1. users

Stores registered system users.

Column	Type	Constraints	Default
id	SERIAL	Primary key	
username	TEXT	Unique, Not Null	
hashed_password	TEXT	Not Null	
email	TEXT	Unique	
is_active	BOOLEAN		true
created_at	TIMESTAMP		now()
2. aes_keys

Holds AES encryption keys, managed and rotated securely.

Column	Type	Constraints	Default
id	SERIAL	Primary key	
key_label	VARCHAR(64)	Not Null	
key_value	TEXT	Not Null	
is_active	BOOLEAN		true
created_at	TIMESTAMP		now()
rotated_at	TIMESTAMP		NULL
3. key_logs

Tracks all key usage and rotation events.

Column	Type	Constraints	Default
id	SERIAL	Primary key	
key_id	INT	FK → aes_keys.id	
timestamp	TIMESTAMP		now()
action	VARCHAR(50)	Not Null	
details	TEXT		NULL
4. entries

Used for general text or data entries associated with users.

Column	Type	Constraints	Default
id	SERIAL	Primary key	
user_id	INT	FK → users.id	ON DELETE CASCADE
title	VARCHAR(255)		
content	TEXT		
created_at	TIMESTAMP		now()
5. orders

Stores order-level data or GUI-submitted records.

Column	Type	Constraints	Default
id	SERIAL	Primary key	
user_id	INT	FK → users.id	ON DELETE SET NULL
description	TEXT	Not Null	
status	VARCHAR(50)		'pending'
created_at	TIMESTAMP		now()
6. order_logs

Maintains history of user or system actions related to orders.

Column	Type	Constraints	Default
id	SERIAL	Primary key	
order_id	INT	FK → orders.id	ON DELETE CASCADE
timestamp	TIMESTAMP		now()
action	VARCHAR(50)	Not Null	
details	TEXT		
7. logs

Global system logs, recording API calls, GUI events, and internal operations.

Column	Type	Constraints	Default
id	SERIAL	Primary key	
timestamp	TIMESTAMP		now()
level	VARCHAR(20)		'INFO'
component	VARCHAR(50)		NULL
message	TEXT		
user_id	INT	FK → users.id	
🧩 Entity Relationship Model
users ──┬──< entries
        ├──< orders ──┬──< order_logs
        │              └──< logs
        └──< logs

aes_keys ──┬──< key_logs

🚀 FastAPI Application Overview

Endpoints (planned/implemented):

Endpoint	Method	Description
/register	POST	Create a new user
/login	POST	Authenticate and return JWT
/entries	GET/POST	List or create entries
/orders	GET/POST	Manage orders
/logs	GET	View activity logs
/aes/rotate	POST	Rotate AES key and log event

All API routes are secured by JWT authentication middleware. Sensitive operations (like /aes/rotate) are further restricted to admin roles.

🖥️ Graphical User Interface (GUI)

Framework: Tkinter
Goal: Provide an intuitive, local client that interacts with the FastAPI backend.

Functional Requirements:

Authentication Panel: Username/password login → JWT token retrieval.

Order Entry Form: Fields for description, status, and submission to /orders.

Data Display: Retrieve and show recent entries or orders via REST API.

Logging Display: Show application and user logs in a scrollable frame.

Encryption Toggle (Future): Option to encrypt entry data using the active AES key.

GUI Architecture:

gui.py runs as a multithreaded Tkinter client.

Uses the requests library for REST communication.

Stores tokens locally (in-memory) for session-limited use.

Designed to run alongside Jupyter environments for hybrid research/development workflows.

🛡️ Security Model
Feature	Description
AES Encryption	Server-side symmetric encryption for sensitive fields.
JWT Authentication	Session tokens signed by JWT_SECRET.
Master Key	Used for encrypting AES keys before database persistence.
Salted Password Hashing	Bcrypt or Blowfish (crypt() with gen_salt('bf')).
Audit Logging	Each action logged in logs, key rotations in key_logs.
🚢 Deployment Notes

Initialize Database:

psql -U postgres -d jupyter_db -f schema.sql


Start Backend:

uvicorn main:app --host 127.0.0.1 --port 8000 --reload


Run GUI:

python gui.py


Access Swagger Docs:

http://127.0.0.1:8000/docs
# FastAPI_Server
FastAPI_Server
