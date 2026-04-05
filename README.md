# TaskFlow

TaskFlow is a full-stack project management app.

- Frontend: React + Vite
- Backend: FastAPI + SQLAlchemy
- Database: PostgreSQL

## 1) What was done in your Linux setup

1. Removed the old Windows virtual environment from backend (it had Scripts/Lib structure).
2. Created a Linux virtual environment and used it for backend packages.
3. Installed backend dependencies from backend/requirements.txt.
4. Updated backend code to load variables from backend/.env automatically.
5. Started PostgreSQL using Docker (because local PostgreSQL and npm were not available on host).
6. Started backend on port 8000.
7. Started frontend on port 5173 using a Node Docker container.
8. Verified both endpoints responded successfully.

## 2) General commands you should know

### Create a Linux venv
python3 -m venv .venv

Explanation:
Creates an isolated Python environment in .venv.

### Activate venv
source .venv/bin/activate

Explanation:
Switches shell to use Python/pip from .venv.

### Install Python requirements
pip install -r requirements.txt

Explanation:
Installs all packages listed in requirements.txt.

### Run FastAPI app
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

Explanation:
Starts backend API on port 8000.

### Run React/Vite app
npm install
npm run dev -- --host 0.0.0.0 --port 5173

Explanation:
Installs frontend packages and runs dev server on port 5173.

## 3) Exact project commands used

Project root:
/media/arham/90E2383BE238283C/Data/Programs/React/check1

### Backend environment setup
cd /media/arham/90E2383BE238283C/Data/Programs/React/check1
python3 -m venv .venv
source .venv/bin/activate
pip install -r backend/requirements.txt

Explanation:
Creates and activates one Linux venv at project root, then installs backend dependencies.

### Backend run command
cd /media/arham/90E2383BE238283C/Data/Programs/React/check1/backend
/media/arham/90E2383BE238283C/Data/Programs/React/check1/.venv/bin/python -m uvicorn app.main:app --host 0.0.0.0 --port 8000

Explanation:
Runs backend using the project root venv interpreter explicitly.

### PostgreSQL with Docker
cd /media/arham/90E2383BE238283C/Data/Programs/React/check1
docker run -d --name taskflow-postgres -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD= -e POSTGRES_DB= -p 5432:5432 postgres:16

Explanation:
Starts PostgreSQL container with:
- DB user: postgres
- DB password: 
- DB name: taskflow_db
- Host port 5432 mapped to container port 5432

If container already exists:
docker start taskflow-postgres

### Frontend with Docker (Node on container)
cd /media/arham/90E2383BE238283C/Data/Programs/React/check1
docker rm -f taskflow-frontend
docker run -d --name taskflow-frontend -p 5173:5173 -v /media/arham/90E2383BE238283C/Data/Programs/React/check1/frontend:/app -w /app node:20 sh -c "npm install && npm run dev -- --host 0.0.0.0 --port 5173"

Explanation:
Runs frontend in a Node container and maps app URL to localhost:5173.

## 4) Why Docker PostgreSQL was used instead of local install

Docker was chosen because:
1. Local PostgreSQL client/service was not present in the Linux host environment.
2. Non-interactive sudo install was not available in the current session.
3. Docker was available and working, which gives a fast, reproducible DB setup.
4. Your backend requires PostgreSQL features (for example ARRAY type in models), so Docker PostgreSQL matches production behavior better than switching to SQLite.

## 5) How the PostgreSQL container was created and used

### Creation
Command used:
docker run -d --name taskflow-postgres -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD= -e POSTGRES_DB=taskflow_db -p 5432:5432 postgres:16

### Usage by backend
The backend reads DATABASE_URL from backend/.env:
DATABASE_URL=postgresql://postgres:@localhost:5432/taskflow_db

Backend then connects to localhost:5432, which is mapped to the running Docker container.

## 6) Daily startup workflow (recommended)

Run these in order whenever you start the project.

### Step A: Start database
cd /media/arham/90E2383BE238283C/Data/Programs/React/check1
docker start taskflow-postgres

### Step B: Start backend
cd /media/arham/90E2383BE238283C/Data/Programs/React/check1
source .venv/bin/activate
cd backend
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

### Step C: Start frontend
If npm is installed on host:
cd /media/arham/90E2383BE238283C/Data/Programs/React/check1/frontend
npm install
npm run dev -- --host 0.0.0.0 --port 5173

If npm is not installed on host, use Docker:
cd /media/arham/90E2383BE238283C/Data/Programs/React/check1
docker start taskflow-frontend

If taskflow-frontend does not exist yet, create it once with:
docker run -d --name taskflow-frontend -p 5173:5173 -v /media/arham/90E2383BE238283C/Data/Programs/React/check1/frontend:/app -w /app node:20 sh -c "npm install && npm run dev -- --host 0.0.0.0 --port 5173"

## 7) URLs to verify app

- Frontend: http://localhost:5173
- Backend API: http://localhost:8000
- Backend Swagger docs: http://localhost:8000/docs

## 8) Stop commands

Stop frontend container:
docker stop taskflow-frontend

Stop postgres container:
docker stop taskflow-postgres

Stop backend server:
Press Ctrl+C in backend terminal.

## 9) Quick health checks

Check backend:
curl http://localhost:8000/

Expected response:
{"message":"Welcome to TaskFlow API"}

Check frontend headers:
curl -I http://localhost:5173/

Expected: HTTP/1.1 200 OK
