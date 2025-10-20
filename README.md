### Group Chat + LLM Bot — Setup and Run

This project is a simple group chat with an LLM bot. The backend is FastAPI + SQLAlchemy (async) with MySQL, and the frontend is served as static files by the backend.

### Prerequisites
- **Python**: 3.10+ (tested with 3.12)
- **MySQL/MariaDB** running locally
- macOS users: having Homebrew helps (`brew services start mysql`)

### 1) Database
Start MySQL, then either rely on auto-creation or create the DB and user manually.

Auto-creation (recommended): the app will create tables on startup. It expects this default URL unless overridden:

```bash
# Default used by backend if DATABASE_URL is not set
mysql+asyncmy://chatuser:chatpass@localhost:3306/groupchat
```

Manual (optional):

```bash
# Start MySQL (pick one)
brew services start mysql
# or
mysql.server start

# Create DB/user via provided SQL
mysql -u root -p < groupchat_app_src/sql/schema.sql

# Check DBs and tables
mysql -u root -p -e "SHOW DATABASES;"
mysql -u root -p -D groupchat -e "SHOW TABLES;"
```

### 2) Backend setup (Conda environment)
From the backend directory:

```bash
cd /Users/nitzansaar/Desktop/EE542/lab8/groupchat_app_src/backend
conda create -n ee542-chat python=3.12 -y
conda activate ee542-chat
pip install -r requirements.txt
```

Notes:
- Always activate this Conda env before running the app.
- bcrypt is pinned in `requirements.txt` for compatibility.

### 3) Optional environment variables
Create `groupchat_app_src/backend/.env` to override defaults:

```bash
# DB (defaults shown)
DATABASE_URL=mysql+asyncmy://chatuser:chatpass@localhost:3306/groupchat

# Auth
JWT_SECRET=change_me
JWT_EXPIRE_MINUTES=43200

# LLM endpoint (optional; OpenAI-compatible server)
LLM_API_BASE=http://localhost:8001/v1
LLM_MODEL=llama-3-8b-instruct
LLM_API_KEY=
```

### 4) Run the backend
```bash
cd /Users/nitzansaar/Desktop/EE542/lab8/groupchat_app_src/backend
conda activate ee542-chat
uvicorn app:app --host 0.0.0.0 --port 8000
```

Visit `http://localhost:8000`. Use the UI to Sign Up or Log In.

### 5) Usage notes
- Passwords must be ≤ 72 characters (bcrypt limit). Longer passwords return HTTP 400.
- On first run, tables are auto-created.
- Messages containing `?` trigger an LLM bot reply if `LLM_API_BASE` is configured.

### Troubleshooting
- Port already in use on 8000:
```bash
pkill -f "uvicorn app:app"
```
Then re-run the uvicorn command above (with the Conda env activated).

- 405 on /api/signup: ensure you’re hitting `http://localhost:8000/api/...` with the backend running here, not a different port.

- Bcrypt errors or 500 on signup:
  - Ensure you activated the Conda env: `conda activate ee542-chat`.
  - Reinstall deps inside that env: `pip install -r requirements.txt`.
  - Keep passwords ≤ 72 characters.

- Check data in MySQL:
```bash
mysql -u root -p -D groupchat -e "SELECT id, username, created_at FROM users;"
mysql -u root -p -D groupchat -e "SELECT COUNT(*) FROM messages;"
```

### API quick test (without UI)
```bash
curl -X POST http://localhost:8000/api/signup \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"secret123"}'

curl -X POST http://localhost:8000/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"secret123"}'
```
