# Course Goals (Dockerized MERN Demo)

Minimal MERN-style example split into a Node/Express + MongoDB backend and a React frontend. The app lets you create and delete simple "goal" items.

## Structure

```
backend/   -> Express API + Mongoose ([backend/app.js](backend/app.js))
frontend/  -> React UI ([frontend/src/App.js](frontend/src/App.js))
```

Key files:

- Backend server: [backend/app.js](backend/app.js)
- Mongoose model: [backend/models/goal.js](backend/models/goal.js)
- Frontend root component: [frontend/src/App.js](frontend/src/App.js)
- Goal list components: [frontend/src/components/goals/CourseGoals.js](frontend/src/components/goals/CourseGoals.js), [frontend/src/components/goals/GoalItem.js](frontend/src/components/goals/GoalItem.js), [frontend/src/components/goals/GoalInput.js](frontend/src/components/goals/GoalInput.js)
- UI components: [frontend/src/components/UI/Card.js](frontend/src/components/UI/Card.js), [frontend/src/components/UI/ErrorAlert.js](frontend/src/components/UI/ErrorAlert.js), [frontend/src/components/UI/LoadingSpinner.js](frontend/src/components/UI/LoadingSpinner.js)
- Logging output: backend/logs/access.log (created via morgan in [backend/app.js](backend/app.js))

## Backend (Express + MongoDB)

Defined in [backend/app.js](backend/app.js). Connects to MongoDB at:

```
mongodb://root:root@mongodb:27017/course-goals?authSource=admin
```

Model: [backend/models/goal.js](backend/models/goal.js)

CORS is open (Access-Control-Allow-Origin: \*).

### Endpoints

- GET /goals  
  Returns: `{ goals: [{ id, text }, ...] }`
- POST /goals  
  Body: `{ "text": "My goal" }`  
  Returns: `{ message, goal: { id, text } }`
- DELETE /goals/:id  
  Returns: `{ message }`

Validation: Rejects empty text (422). Errors -> 500 JSON message.

### Logging

Requests logged to `backend/logs/access.log` via morgan ("combined" format).

## Frontend (React)

Entry: [frontend/src/index.js](frontend/src/index.js)  
Root component: [frontend/src/App.js](frontend/src/App.js)

Behavior:

- Loads goals on mount (GET http://localhost/goals)
- Adds goals (POST)
- Deletes goals (DELETE)
- Displays error via [frontend/src/components/UI/ErrorAlert.js](frontend/src/components/UI/ErrorAlert.js)

State:

- loadedGoals
- isLoading
- error

UI Pieces:

- Input form: [frontend/src/components/goals/GoalInput.js](frontend/src/components/goals/GoalInput.js)
- List: [frontend/src/components/goals/CourseGoals.js](frontend/src/components/goals/CourseGoals.js)
- Item: [frontend/src/components/goals/GoalItem.js](frontend/src/components/goals/GoalItem.js)

Spinner component present ([frontend/src/components/UI/LoadingSpinner.js](frontend/src/components/UI/LoadingSpinner.js)) but not yet used in App.

## Running with Docker

Build images (run from each folder):

```
# Backend
cd backend
docker build -t goals-backend .

# Frontend
cd ../frontend
docker build -t goals-frontend .
```

Run containers (example):

```
# MongoDB (simple dev instance)
docker run -d --name mongodb -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=root mongo

# Backend (expects Mongo hostname "mongodb")
docker run -d --name goals-backend --link mongodb:mongodb -p 80:80 goals-backend

# Frontend (CRA dev server on 3000)
docker run -d --name goals-frontend --link goals-backend:backend -p 3000:3000 goals-frontend
```

Access UI: http://localhost:3000  
API base (from frontend code): http://localhost/goals (so ensure backend published on host port 80)

If you map backend differently (e.g. 8080), update fetch URLs in [frontend/src/App.js](frontend/src/App.js).

## Local (Without Docker)

Backend:

```
cd backend
npm install
# Ensure local MongoDB with user/pass root/root or adjust connection string in app.js
npm start
```

Frontend (separate terminal):

```
cd frontend
npm install
npm start
```

## Scripts

Backend `package.json`:

- `start`: nodemon app.js

Frontend `package.json`:

- `start`: react-scripts start
- `build`: react-scripts build
- `test`: react-scripts test

## Notes / Improvements

- Connection string is hardcoded; Dockerfile sets ENV vars but they are unused. Consider reading `process.env.MONGODB_USERNAME` / `MONGODB_PASSWORD`.
- Add a docker-compose.yml to orchestrate Mongo + backend + frontend.
- Use the existing LoadingSpinner component in [frontend/src/components/UI/LoadingSpinner.js](frontend/src/components/UI/LoadingSpinner.js) while fetching.
- Centralize API base URL via environment variable (e.g. `REACT_APP_API_BASE`).
- Add error reset and optimistic UI updates.
- Add schema validation (e.g. required text) in [backend/models/goal.js](backend/models/goal.js).
- Add tests for components and endpoints.

## License

Educational demo; original author attribution present in
