# My Goals (Dockerized MERN Demo)

Minimal MERN-style example: Node/Express + MongoDB backend & React frontend to create/delete simple "goal" items.

## Structure

```
backend/          Express API (Mongo + Mongoose)
frontend/         React app
docker-compose.yaml
env/              Environment files (mongo.env, backend.env)
```

## Backend (Express + MongoDB)

File: backend/app.js  
Default connection (hardcoded):

```
mongodb://root:root@mongodb:27017/course-goals?authSource=admin
```

Model: backend/models/goal.js

### Endpoints

- GET /goals -> { goals: [{ id, text }] }
- POST /goals { "text": "My goal" } -> { message, goal }
- DELETE /goals/:id -> { message }

Validation: empty text => 422 JSON. Other errors => 500 JSON.  
Logging: backend/logs/access.log via morgan (combined).

## Frontend (React)

Entry: frontend/src/index.js  
Root: frontend/src/App.js  
Fetch base: http://localhost/goals (expects backend bound to host port 80)

State handled: loadedGoals, isLoading, error  
Components:

- Goals: frontend/src/components/goals/{CourseGoals.js,GoalItem.js,GoalInput.js}
- UI: frontend/src/components/UI/{Card.js,ErrorAlert.js,LoadingSpinner.js}

## Running with Docker Compose

Prereq: Docker + Compose plugin.

docker-compose.yaml defines:

- mongodb (persistent volume: mongoData)
- backend (port 80:80, code bind-mounted for live reload)
- frontend (port 3000:3000, src bind-mounted for hot reload)

Environment files (create/edit):
env/mongo.env:

```
MONGO_INITDB_ROOT_USERNAME=root
MONGO_INITDB_ROOT_PASSWORD=root
```

env/backend.env (example placeholders):

```
MONGODB_USERNAME=root
MONGODB_PASSWORD=root
MONGODB_DATABASE=course-goals
```

(backend currently still uses hardcoded string—adjust code to read these if desired)

Start (build + run):

```
docker compose up --build
```

Stop:

```
docker compose down
```

Stop + remove volumes (Mongo data + logs):

```
docker compose down -v
```

Access:

- Frontend: http://localhost:3000
- API: http://localhost/goals

### Custom Image Names

In docker-compose.yaml set:

```
image: your-dockerhub-user/goals-backend:1.0.0
image: your-dockerhub-user/goals-frontend:1.0.0
```

Build:

```
docker compose build
```

Login & push:

```
docker login
docker compose push
```

Retag for new versions (e.g. :1.0.1) then rebuild & push.

## Manual Docker (Without Compose)

Backend:

```
cd backend
docker build -t goals-backend .
docker run -d --name mongodb -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=root mongo
docker run -d --name goals-backend --link mongodb:mongodb -p 80:80 goals-backend
```

Frontend:

```
cd frontend
docker build -t goals-frontend .
docker run -d --name goals-frontend --link goals-backend:backend -p 3000:3000 goals-frontend
```

## Local Dev (No Docker)

Backend:

```
cd backend
npm install
npm start
```

(Adjust Mongo URI in backend/app.js to local instance if needed.)
Frontend:

```
cd frontend
npm install
npm start
```

## Scripts

Backend: npm start (nodemon)  
Frontend: npm start | npm run build | npm test
