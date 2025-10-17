# Task Management System - DevOps Project

**TODO** Käännetään tämä myös suomenkieliseksi README_fi.md tiedostoon.

A full-stack task management application built for learning DevOps practices including CI/CD pipelines, containerization, and cloud deployment.

## Features

- User authentication (Register/Login)
- Create, read, update, delete tasks
- Task status management (Todo, In Progress, Done)
- Task priority levels (Low, Medium, High)
- Due dates
- Filter and search functionality

## Technology Stack

**Backend:**
- Node.js with Express
- TypeScript
- Sequelize ORM
- PostgreSQL database
- JWT authentication

**Frontend:**
- React 19 & Vite
- TypeScript
- React Router
- Axios

## Architecture

```mermaid
flowchart LR
   subgraph Frontend["Frontend"]
      FE["React App"]
   end

   subgraph Backend["Backend"]
      BE["API Server (Node + Express)"]
      Auth[(JWT)]
   end

   subgraph Data["Data & Persistence"]
      DB[(PostgreSQL)]
   end

   FE -->|API calls| BE
   BE -->|HTTP responses| FE

   FE -->|Authorization: Bearer token| BE
   BE -->|issues JWT on auth| FE

   BE -->|Read/Write| DB
   DB -->|Query results| BE

   BE -->|issue/verify token| Auth
   Auth -->|token used by| BE
 ```

# Project Tasks

### Prerequisites

- Node.js 22+ and npm
- Docker and Docker Compose
- Git

### Default Ports

- Frontend: `5173`
- Backend: `5000`
- Database: `5432`

### Tips for Getting Started
- **Start simple:** Begin by setting up a basic GitHub Actions workflow that runs on push and pull requests.
- **Incremental approach:** Add one CI/CD step at a time (e.g., first linting, then testing, then Docker build).
- **Use official actions:** Leverage pre-built GitHub Actions for Node.js, Docker, and CodeQL to simplify your workflow.
- **Test locally:** Make sure your tests and Docker builds work locally before automating them in CI/CD.
- **Read documentation:** Refer to [GitHub Actions docs](https://docs.github.com/en/actions) and [Docker docs](https://docs.docker.com/) for examples and troubleshooting.
- **Monitor pipeline runs:** Check the Actions tab in your repository for logs and troubleshooting information.
- **Iterate and improve:** Refine your workflow as you discover new requirements or issues.

### References
- Workflow syntax for Github actions: https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax
- The following Docker workshop provides clear instructions that are helpful for completing the docker tasks in this project: https://docs.docker.com/get-started/workshop
- The Render.com documentation helps in the deployment task: https://render.com/docs

## Task 1. Run Task manager Locally

 It is recommended to run the application locally before starting CI/CD to quickly identify and resolve issues in the development environment. Running locally allows you to verify basic functionality, catch errors early, and ensure dependencies are correctly configured.

 This step also helps prevent unnecessary CI/CD pipeline failures.

### 1. Clone repository and install dependencies
```bash
git clone <repo_url>
cd <directory_name>
npm run install:all
```
### 2. Run PostgreSQL in Docker:
Powershell:
```bash
docker run -d `
  --name postgres-local `
  -e POSTGRES_DB=taskmanagement `
  -e POSTGRES_USER=postgres `
  -e POSTGRES_PASSWORD=postgres `
  -p 5433:5432 `
  postgres:18-alpine
```
Linux or MacOS
```bash
docker run -d \
  --name postgres-local \
  -e POSTGRES_DB=taskmanagement \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -p 5433:5432 \
  postgres:18-alpine
```
### 3. .env files
Copy `.env.example` to `.env` both in frontend and backend.
This creates a real `.env` file that your app will use.

#### Backend .env
```
NODE_ENV=development
PORT=5000
DATABASE_URL=postgresql://postgres:postgres@localhost:5433/taskmanagement
JWT_SECRET=your-super-secret-jwt-key-change-in-production
JWT_EXPIRES_IN=7d
FRONTEND_URL=http://localhost:5173
# Set to true during initial production deployment to sync the database
# Switch back to false after the database has been synchronized
DB_SYNC=false
```
#### Frontend .env
```
VITE_API_URL=http://localhost:5000/api
```

### 4. Run backend
```bash
npm run build:backend
npm run dev:backend
```
### 5. Run frontend
```bash
npm run dev:frontend
```

The application will be available at:
- Frontend: http://localhost:5173
- Backend: http://localhost:5000
- Database: localhost:5432

## Task 2. Implement CI pipeline

Your goal is to implement a CI pipeline for the Task Management System:

> **Tip:** This project uses [npm workspaces](https://docs.npmjs.com/cli/v10/using-npm/workspaces), you can run commands in specific subdirectories by setting the [`working-directory`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsworking-directory) property in your GitHub Actions job step. For example:
   > ```yaml
   > - name: Install dependencies
   >   working-directory: ./backend
   >   run: npm install
   > ```
   > 
   > Review the `package.json` files in the root directory to see the available scripts for each part of the CI pipeline.

1. **Set Up a CI Workflow:**
   - Use GitHub Actions to automate your workflow.
   - The pipeline should run on every push and pull request to the main branch.

2. **Automate Testing and Linting:**
   - Configure the pipeline to run backend and frontend tests.
   - Add a linter step to check code quality.

   **HUOM** Coverage testin ajaminen ja tuloksen vieminen artifactina voisi olla kiva lisä. Täytyy ihmetellä tuota.

## Task 3. Security 
   1. **Static code analysis & dependencies**
      - Integrate CodeQL analysis in the CI/CD pipeline to automatically scan for vulnerabilities.
      - Add steps to check for outdated or vulnerable dependencies using tools like `npm audit`.


## Task 4. Docker Integration
   - Build Docker images for the backend and frontend as part of the pipeline.
   - Optionally, push images to a container registry.

## Task 5. Deployment to Render.com

You can choose either **Path 1 (Manual deployment & CD workflow)** or **Path 2 (Deployment using containers)** for deploying your application to Render.com. 

### Path 1: Manual deployemnt & CD workflow
   1. **PostgreSQL database** 
      - Create PostgreSQL database to render.com
      - You should remember database URL (Internal Database URL) for the next step 2.
   2. **Backend** 
      - Create Web Service for the backend
         - Root directory: `backend`
         - Build command: `npm install && npm run build`
         - Start command: `npm start`
         - Set the environment variables:
         ```
         NODE_ENV=production
         PORT=5000
         DATABASE_URL=<INTERNAL DATABASE URL>
         JWT_SECRET=generate a random secure string
         JWT_EXPIRES_IN=7d
         FRONTEND_URL=<SET THIS AFTER FRONTEND SETUP>
         DB_SYNC=true
         ```

      After deployment, review the render.com backend deployment logs for the following messages. These confirm a successful database connection and that the tables have been created.
      ```
      Database connection established successfully.
      Database models synchronized.
      ```
      
      Check the backend health endpoint: `https://your-backend.onrender.com/api/health`

   3. **Frontend**
      - Create Static site for the frontend
         - Root Directory: `frontend`
         - Build Command: `npm install && npm run build`
         - Publish Directory: `dist`
         - Set the environment variables
         ```
         VITE_API_URL=<BACKEND URL>/api
         ```
   4. **Update CI/CD workflow**
      - Disable automatic deployment for both backend and frontend services in Render.com.
      - Update your CI workflow from Task 2 to include a CD (Continuous Deployment) step.
      - Configure deployment to trigger only when a new release tag is pushed.

> **Tip:** Render.com provides **Blueprints**, which are YAML files that define your entire infrastructure as code (IaC). Blueprints allow you to version, review, and automate the setup of services, databases, environment variables, and more directly from your repository.  
>  
> With Blueprints, you can:
> - Describe all resources (web services, static sites, databases) in a single file.
> - Automate deployments and updates by pushing changes to your repository.
> - Ensure consistent environments across teams and deployments.
>  
> To use Blueprints for deployment:
> 1. Create a `render.yaml` file in your repository root.
> 2. Define your services, build commands, environment variables, and database configuration in the file.
> 3. Connect your repository to Render.com and enable Blueprint deployment.
> 4. Render will automatically provision and update resources based on your Blueprint file.
>  
> Learn more: [Render Blueprints documentation](https://render.com/docs/infrastructure-as-code)

### Path 2: Deployment using containers

TÄMÄ OLISI VAIHTOEHTOINEN POLKU. TÄTÄ EN OLE ITSE TESTANNUT

This option deploys both backend and frontend as Docker containers on Render.com.

1. **Create Dockerfiles**
   - Create a `Dockerfile` in both `backend` and `frontend` directories if you haven't done these in Task 4.

2. **Build and Test Locally**
   - Build and test docker files locally

3. **Push Images to Registry**
   - Use Docker Hub
  
4. **Create Services on Render.com**
   - Create a new **Web Service** for backend:
     - Select "Deploy an existing Docker image".
     - Set image source to your registry.
     - Set environment variables.
   - Create a new **Web Service** for frontend:
     - Use the frontend image.
     - Set environment variables.

5. **Configure Database**
   - Create a PostgreSQL database on Render.com.
   - Update backend environment variables with the internal database URL.

6. **Update CI/CD Workflow**
   - Add steps to build and push Docker images on release/tag.
   - Trigger deployment on Render.com after pushing new images.
  
**References:**
- [Render Docker Deployment Docs](https://render.com/docs/docker)

## Task 6. Monitoring
- SOMETHING SIMPLE HERE: Bäkkärissä on health check url. Lokituksen monitorointi? Tämä vaatisi varmaan lisäyksi lokitukseen sovelluksessa. Jätetäänkö tämä osuus pois lopputyöstä??

**TODO** PISTEYTYS TASKEILLE

## Deliveries & Submission
For successful completion of the project, submit the following:

1. **Source Code Repository**
   - Complete codebase for backend and frontend.
   - Include all configuration files.

2. **CI/CD Workflow**
   - GitHub Actions workflow file(s) implementing CI (testing, linting, security checks) and CD (deployment steps).

3. **Docker Integration**
   - Dockerfiles for backend and frontend.
   - Instructions or scripts for building and running containers locally.

4. **Deployment**
   - Render.com service URLs for backend and frontend.
   - Description of environment variable setup and deployment process.

5. **Monitoring**
   - Description of implemented health checks and logging/monitoring approach.

6. **Documentation**
   - Updated `README.md` with setup, usage, and deployment instructions.
   - Brief summary of what was implemented, challenges faced, and lessons learned. 