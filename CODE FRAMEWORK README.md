# AI-ASSIGNMENT-GRADING-SYSTEM

## Table of Contents
- [AI-ASSIGNMENT-GRADING-SYSTEM](#ai-assignment-grading-system)
  - [Table of Contents](#table-of-contents)
  - [Architecture](#architecture)
    - [Component Descriptions](#component-descriptions)
  - [Diagram Images and Descriptions](#diagram-images-and-descriptions)
    - [Entity Relationship Diagram of Assignment Grading System Database](#entity-relationship-diagram-of-assignment-grading-system-database)
    - [Assignment Creation Process](#assignment-creation-process)
    - [Azure Components Functional Diagram](#azure-components-functional-diagram)
    - [Component Descriptions](#component-descriptions-1)
    - [AI Grading System Overview](#ai-grading-system-overview)
    - [Component Descriptions](#component-descriptions-2)
    - [Legend](#legend)
    - [Moodle LMS Integration Process](#moodle-lms-integration-process)
    - [Integration Steps and Component Descriptions](#integration-steps-and-component-descriptions)
    - [Automated Section Mapping](#automated-section-mapping)
    - [Process Overview and Component Descriptions](#process-overview-and-component-descriptions)
  - [Documentation](#documentation)
  - [Useful links:](#useful-links)
  - [Prerequisites:](#prerequisites)
  - [Setting up the environment:](#setting-up-the-environment)
  - [Runing the app locally:](#runing-the-app-locally)
    - [Start the frontend and backend individually:](#start-the-frontend-and-backend-individually)
    - [Start the project as a docker compose:](#start-the-project-as-a-docker-compose)
    - [Start each container separately:](#start-each-container-separately)
    - [If you want to run evaluations and need any of the files in the `/data` folder you must download them from the `/data` folder from the file share.](#if-you-want-to-run-evaluations-and-need-any-of-the-files-in-the-data-folder-you-must-download-them-from-the-data-folder-from-the-file-share)
    - [If yow want to run evaluations and need any of the csv files you must download them from `/csv` folder from the file share.](#if-yow-want-to-run-evaluations-and-need-any-of-the-csv-files-you-must-download-them-from-csv-folder-from-the-file-share)
  - [Deploying the application:](#deploying-the-application)
  - [Create Sample Assignment Definitions](#create-sample-assignment-definitions)
  - [Database integration with alembic](#database-integration-with-alembic)
    - [Configuration](#configuration)
    - [Running Migrations](#running-migrations)
      - [To verify the database migration state:](#to-verify-the-database-migration-state)
      - [Apply New Migrations](#apply-new-migrations)
      - [Roll Back Migrations](#roll-back-migrations)
      - [Generate a New Migration](#generate-a-new-migration)
  - [Running tests](#running-tests)
    - [Prerequisite](#prerequisite)
    - [Make sure you installed all the dependencies:](#make-sure-you-installed-all-the-dependencies)
    - [Setup PYTHONPATH](#setup-pythonpath)
    - [Run the tests](#run-the-tests)
    - [Notes](#notes)
- [\*\*Moodle Integration \*\*](#moodle-integration-)
  - [**1. Start Moodle Locally**](#1-start-moodle-locally)
  - [**2. Enable Web Services**](#2-enable-web-services)
  - [**3. Create an External Service**](#3-create-an-external-service)
  - [**4. Enable Required Web Service Functions**](#4-enable-required-web-service-functions)
  - [**5. Generate and Assign a Token**](#5-generate-and-assign-a-token)
  - [**6. Create a Course and Assignment**](#6-create-a-course-and-assignment)
  - [**7. Create and Enroll a Student**](#7-create-and-enroll-a-student)
  - [**8. Submit an Assignment for the Student**](#8-submit-an-assignment-for-the-student)
  - [**9. Trigger Grading**](#9-trigger-grading)

## Architecture

The architecture diagram below displays the architecture of the Custom Gen AI Application used for assessing student assignments. It highlights how papers are processed and evaluated through the system's various components, leveraging AI for automated assessment.

### Component Descriptions

1. **Custom WebGUI**
   - Acts as the user interface through which teachers can upload student papers and retrieve assessment results.
   - Provides a streamlined process for managing assignments and viewing AI-generated feedback.
   - Interfaces with the ingestion component to initiate data processing and section mapping.

2. **Ingestion Component**
   - Handles the intake of uploaded papers, breaking down the bibliography into sections and mapping these to evaluation criteria.
   - Stores processed sections and criteria mappings in the PostgreSQL database for further use in assessments.

3. **PostgreSQL DB**
   - Serves as the storage solution for sections, criteria mappings, and future assessment results.
   - Facilitates data retrieval for the orchestrator, ensuring that assessments are conducted using up-to-date and relevant information.

4. **Orchestrator**
   - A Python application deployed in Docker that orchestrates the assessment process by interfacing with the configurable LLM.
   - Retrieves necessary sections and criteria from the database and coordinates the assessment workflow.
   - Returns assessment results in JSON format to the Custom WebGUI, where they can be accessed by teachers.

5. **Configurable LLM (Large Language Model)**
   - Utilized by the orchestrator to perform assessments with advanced AI capabilities, such as those provided by ChatGPT 4 and Claude Sonnet 3.5.
   - Assesses student papers against predefined criteria, generating intelligent, objective feedback and scores.

![Architecture](diagrams/images/architecture.png)

## Diagram Images and Descriptions

### Entity Relationship Diagram of Assignment Grading System Database

Below you can find the Entity Relationship Diagram (ERD) which illustrates the database schema of the AI Assignment Grading System and a short description of the role of each entity. It showcases the various entities involved in the grading process and their interactions within the system.

1. **Document**
   The workbook file that contains the information the student should study in order to pass the exam.
2. **AssignmentDefinition**
    The defintion of the assignment, it includes the title, the unit number, academic year and the description of the assignment, alongisde with the moodle id for the moodle integration.
3. **Part** Usually an assignment definition has one or more parts that the student should address. It contains an unique id, a name (can be Part 1, Part A, etc,), a short description of the assignment for this particular part of the assignment definition and the assignment definition id to link to the assignment definition.
4. **PartSection** this holds the relationship between the workbook sections for each part. This process is similar to the section criteria mapping and is explained in more details in that section. 
5. **LearningObjective** the place holder for the learning objectives for the assignment definition. This will be linked to the criterias entity as each learning objective can have one or more criterias. The criterias as well can be linked to one or more learning objectives.
6. **Section** represents the workbook document sections that will be later mapped to one or more criterias. So when the LLM will grade the student's work for each criteria, we will only feed the necessary sections for that particular criteria.
7. **Criteria** the criterias entity for the learning objective as explained in the learning objective section.
8. **SectionCriteriaMapping** will hold the mapping between the workbook section and the criteria as explained above in the Section entity.
9. **Assignment** represents the student's assignment that will later be graded for that particular assignment defintion.
10. **Student** the entity for the student. It holds relevant information to track each student's assignment.
11. **AssignmentSection** Similar to the assignemnt defintion workbook, the student's assignment will be split into sections which will be later mapped to criterias so the LLM can easily identify which student's sections answer the specific criteria.
12. **AssignmentSectionCriteriaMapping** the mapping between the student's assignment and the criterias as explained above.
13. **Grading** holds relevant information after the LLM graded the student's work like the overall score, general grade, how much compute tokens it took the LLM to grade and more.
14. **CriteriaScore** this entity holds the relevant information for each criteria after the LLM finished grading.
![Entity Relationship Diagram](diagrams/images/Database%20scheme%20and%20relationship%20diagram.png)


### Assignment Creation Process

The assignment creation process diagram illustrates the workflow involved in creating assignments within the AI Assignment Grading System. It showcases the interactions between the teacher, frontend, backend, and database during the process of assignment creation and automated section mapping.

1. **Teacher**
   - The teacher initiates the assignment creation process by accessing the assignment creation page on the frontend system.
   - Enters assignment details including bibliography, assignment text, learning objectives, and criteria.

2. **Frontend**
   - **Access Assignment Creation Page:** Requests and displays the assignment form for the teacher.
   - **Display Assignment Form:** Allows the teacher to enter and submit assignment data.
   - **Confirm Assignment Creation:** Displays confirmation messages to the teacher indicating various stages of the assignment creation process.
   - **Notify Mapping Completion:** Informs the teacher once the automated mapping is complete and ready for review.
   - **Submit Mapping Adjustments:** Facilitates submission of any adjustments the teacher makes to the mapping.
   - **Confirm Assignment is Ready:** Updates the teacher with a final message when the assignment is completed and status is set to "READY".

3. **Backend**
   - **Request Assignment Form:** Fetches the assignment form details from the database as requested by the frontend.
   - **Store Assignment Data:** Saves the initially submitted assignment data to the database with a status of "PENDING".
   - **Perform Automated Mapping:** Executes automated processes to map assignment sections to predefined criteria.
   - **Update Assignment Status:** Modifies the assignment status in the database from "PENDING" to "MAPPED" after automation completes, and to "READY" after teacher adjustments are processed.
   - **Confirm Storage and Updates:** Sends confirmations back to the frontend at various stages to ensure completion status integrity.

4. **Database**
   - Serves as the central repository for storing assignment data and mapping statuses.
   - Supports fetching and updating of assignment information throughout the workflow.
![Assignment Creation Process](diagrams/images/define_assignments_sequence.png)

### Azure Components Functional Diagram

Below you will find the functional diagram which provides an overview of the Azure components involved in the AI Assignment Grading System. It illustrates how user interactions with the frontend web application are processed via container apps, and how they interface with various Azure services.

### Component Descriptions

1. **User**
   - The end-user who accesses and interacts with the frontend web application. This can be a teacher, student, or administrator using the system.

2. **Frontend Web Application**
   - The interface presented to the user. It allows users to perform actions such as creating assignments, viewing submissions, and assessing grades.
   - Communicates directly with the Container App to process user requests and display data retrieved from back-end services.

3. **Container App**
   - A versatile microservice that performs logic transactions, processes data, and connects to Azure services.
   - Facilitates communication between the frontend web application and backend resources such as the database, AI services, and storage.
   - Handles queries, data processing, and response generation for the frontend.

4. **Azure Database for PostgreSQL**
   - A managed relational database service that stores structured data related to assignments, users, grading, and other core functionalities.
   - Processes queries sent by the Container App to manage data retrieval and storage operations.

5. **Azure OpenAI Service**
   - Provides AI capabilities for grading and feedback. It can process language queries and provide intelligent responses and grading suggestions.
   - Connected by the Container App to enhance assignment evaluation processes using AI models.

6. **Azure Storage Account**
   - Facilitates storage and management of files and media assets related to assignments, submissions, and resources.
   - The Container App reads from and writes data to the storage account, managing file operations and ensuring data integrity.

![Azure Components Functional Diagram](diagrams/images/diagram%20of%20architecture%20of%20application%20or%20micro-services.png)


### AI Grading System Overview

This high-level architectural diagram illustrates the flow of data and interactions within the AI Grading System. It highlights the integration between teachers, the web application, learning management systems, and large language models to streamline assignment evaluations.

### Component Descriptions

1. **Teacher**
   - The primary user of the system who utilizes the AI grading system to assess student assignments.
   - Interfaces with the Single-Page App (SPA) to manage assignments and review AI-generated grades.

2. **Single-Page App (SPA)**
   - Developed using JavaScript and Angular, providing comprehensive frontend functionality to teachers.
   - Offers a user-friendly web interface through which teachers interact with the AI Grading System.
   - Communicates asynchronously with the backend via JSON over HTTPS to request data and submit grading decisions.

3. **Web Application**
   - Built with Python and FastAPI, acts as the core backend service.
   - Delivers API endpoints for SPA to call and orchestrates interactions with AI agents for assignment evaluations.
   - Manages communications with both the database and external large language models (LLMs).

4. **Database**
   - A SQL database that stores essential information including user registrations, authentication credentials, and access logs.
   - Allows synchronous data read and write operations through JDBC to ensure data integrity and availability.

5. **Large Language Model (LLM)**
   - Implements advanced AI models such as ChatGPT and Claude for conducting assignment evaluations.
   - Engages asynchronously with the web application to perform natural language processing and generate grading results.

6. **Learning Management System (LMS)**
   - An external system used by Skills4Stem to monitor and track student progress.
   - The AI Grading System uploads evaluation results to the LMS via synchronous/asynchronous XML/HTTP requests.
   - Facilitates seamless assignment distribution to students and consolidates grading feedback.

### Legend
- **Person**: Represents the human users interacting with the system.
- **System**: Denotes the AI Grading System as a whole.
- **Container**: Signifies software components encapsulating different functionalities.
- **External Person/System**: Identifies external users and services interfacing with the AI Grading System.

![AI Grading System Overview](diagrams/images/high_level_architecture.png)


### Moodle LMS Integration Process

This sequence diagram illustrates the steps involved in integrating the Moodle Learning Management System (LMS) with the AI Grading Platform. It demonstrates the interactions between administrators, the AI grading platform, Moodle LMS, and the AI grader to efficiently handle assignments and grading.

### Integration Steps and Component Descriptions

1. **Admin**
   - Configures the course ID and assignment name within the AI Grading Platform, initiating the process for managing assignments through Moodle LMS.

2. **AI Grading Platform**
   - **Save Assignment Definition:** Stores the assignment details configured by the admin for further processing.
   - **Poll for Updates:** Regularly polls Moodle LMS using `core_course_get_updates_since` to check for any updates or changes since the last sync.
   - **Store Last Check Timestamp:** Records the timestamp of the last update check to ensure accurate retrieval of assignment modifications.
   - **Retrieve Assignments:** Uses `mod_assign_get_assignments` to fetch assignment data from Moodle LMS.
   - **Filter Assignments by Name:** Filters the retrieved assignments to match those configured by the admin for grading.
   - **Retrieve Submissions:** Collects student submissions associated with the matched assignments from Moodle LMS.
   - **Send Submissions for Grading:** Sends the collected submissions to the AI Grader for assessment and evaluation.
   - **Store Feedback Files and Grade:** After grading, stores feedback files and grades in Moodle LMS using `mod_assign_save_grade`.

3. **Moodle LMS**
   - Acts as the centralized platform for managing course content and assignments.
   - Provides APIs for fetching assignment and submission updates, facilitating seamless grading integration with the AI Grading Platform.

4. **AI Grader**
   - Processes received submissions and generates grades and feedback using AI models.
   - Ensures accurate and objective evaluation of student work based on predefined criteria.

![Moodle LMS Integration Process](diagrams/images/moodle_integration.png)


### Automated Section Mapping

sequence diagram illustrates the automated section mapping process within the AI Assignment Grading System. It highlights how documents in the bibliography are segmented and mapped to criteria, and details the interactions among system components to ensure sections are stored correctly.

### Process Overview and Component Descriptions

1. **Assignment Creator**
   - Initiates the creation or update of an assignment, prompting the system to start the automated section mapping process.
   - Reviews and finalizes mappings through an interface provided by the GUI.

2. **GUI (Graphical User Interface)**
   - Facilitates user interaction with the system, enabling assignment creators to initiate mapping procedures and review sections linked with criteria.
   - Presents the current mappings and allows users to finalize any adjustments via the frontend platform.

3. **Backend API**
   - Acts as an intermediary between the GUI and other backend services, processing requests related to section mapping and data retrieval.
   - Ensures efficient data flow and communication between components, handling assignment creation and mapping updates.

4. **Section Mapping Service**
   - **Trigger Section Mapping:** Starts the process of automatically dividing documents into sections upon request.
   - Handles loops for each document to segment text and store sections in the database.
   - Further maps sections to criteria based on the assignment contents, storing the links for future reference.
   - Confirms successful storage of all mappings in the database.

5. **Database**
   - Stores sections derived from bibliography documents and links these sections with assignment criteria.
   - Provides access to current mappings and stores finalized mappings, maintaining integrity and consistency of data.

6. **LLM (Large Language Model)**
   - Although not directly detailed in this diagram, LLMs can assist in processing and understanding text sections, potentially optimizing the mapping process.
   - Enhances the accuracy and relevance of section mappings by leveraging AI capabilities to analyze document content.


![Automated Section Mapping](diagrams/images/bibliography_section_mapping.png)


## Documentation

You can check the user guide to see how a user can use the app to create assignments, assignments definitions, grade students, etc.

You can find the user guide inside the "adoc" folder. "adoc/AI Assignment Grading System User Guide.pdf

[UserGuide](adoc/AI%20Assignment%20Grading%20System%20User%20Guide.pdf)

You can find all the relevant documentation like Database scheme and relationship diagram [DatabaseScheme](adoc/Diagram%20of%20architecture%20of%20application%20or%20micro-services.puml), Diagram of architecture [ArchitectureDiagram](adoc/diagram%20of%20architecture%20of%20application%20or%20micro-services.puml) and many more relevant architecture diagrams and documentation inside the "adoc" folder.

We recommend installing the PUML extenstion for Visual Studio Code so you can preview the diagrams directly in Visual Studio Code.

## Useful links:
1. Phase 2 Sharepoint: https://minditro.sharepoint.com/sites/presales/Shared%20Documents/Forms/AllItems.aspx?id=%2Fsites%2Fpresales%2FShared%20Documents%2FSkills4Stem%2FPhase%202&viewid=fa5e03e3%2Dc23f%2D40e3%2Db9bd%2D7e8e2c896d52S

## Prerequisites:
1. Git CLI: https://git-scm.com/book/en/v2/Getting-Started-The-Command-Line
2. Azure CLI: https://learn.microsoft.com/en-us/cli/azure/get-started-with-azure-cli
3. GCloud CLI: https://cloud.google.com/sdk/docs/install
4. SWA CLI: https://azure.github.io/static-web-apps-cli/
5. Claude 3.5 Sonnet: https://console.cloud.google.com/vertex-ai/publishers/anthropic/model-garden/claude-3-5-sonnet
6. Docker: https://www.docker.com/
7. Clone repository: https://minditsoftware@dev.azure.com/minditsoftware/Skills4Stem/_git/ai-assignment-grading-system or git@ssh.dev.azure.com:v3/minditsoftware/Skills4Stem/ai-assignment-grading-system
8. Make sure you get access to the Skills4Stem env file stored in mindit.io Dashlane. Talk to your team members to get access to that.
9. Make sure you get access to the Google Application credentials file (`mindit-ai-playground-9c16b0a8f1a4.json`) stored in mindit.io Dashlane.  Talk to your team members to get access to that.

## Setting up the environment:
Additionally you can run the application from a container
1. Create python virtual environment: `python -m venv <you-env-name-here>`
2. Activate the environment: `./<your-env-name-here>/Scripts/Activate.ps1` or `source <your-env-name-here>/bin/activate` for linux/mac
3. Installing the backend dependencies: pip install -r src/python/requirements.txt
4. Installing the frontend dependencies: ./ui-platform/npm i
5. Copy .env file from Skills4Stem Dashlane to `src/python/components`
6. Copy Google Application Credentials file (mindit-ai-playground-*.json) to `src/python`

## Runing the app locally:
### Start the frontend and backend individually:
1. Create a local database using docker(one time only):
```bash
docker run -d   --name ai-assignment-grading-system-db-test \
  -e POSTGRES_USER=aiassignmentgradingsystemtest \
  -e POSTGRES_PASSWORD=ai-Assignment-grading01test \
  -e POSTGRES_DB=postgres \
  -p 5432:5432 \
  postgres:latest
```
2. Run the alembic migrations
```bash
cd src/python/components

# Verify current alembic state
alembic current

# Run migrations
alembic upgrade head
```
3. Start the backend

```bash
# Go to components folder if not there already
cd src/python/components

# Get .env file from Dashlane (ask a colleague if you don't have access) and put in the components folder
# Also, make sure .env file points to localhost database (both LOCAL_DATABASE_URL and SQLALCHEMY_URL)
# LOCAL_DATABASE_URL=postgresql://aiassignmentgradingsystemtest:ai-Assignment-grading01test@localhost:5432/postgres
# SQLALCHEMY_URL=postgresql://aiassignmentgradingsystemtest:ai-Assignment-grading01test@localhost:5432/postgres

# Start the backend
uvicorn api.main:app --host 0.0.0.0 --port 8000 --reload
```
4. Start the frontend. In a separate terminal:
```bash
# Go to frontend folder
cd ./ui-platform

# Install the npm dependencies (only needed when running for the first time or a new dependency was added)
npm install


# Start the frontend
npm run dev
```

5. Open the frontend in the browser: http://localhost:3000


### Start the project as a docker compose:
1. docker-compose up --build

### Start each container separately:
1. Backend:
  * cd src/
  * docker build -t aibackend .
  * docker run -p 80:80 aibackend
2. Frontend:
  * cd ui-platform/
  * docker build -t aifrontend .
  * docker run -p 3000:3000 aifrontend
  
### If you want to run evaluations and need any of the files in the `/data` folder you must download them from the `/data` folder from the file share.

### If yow want to run evaluations and need any of the csv files you must download them from `/csv` folder from the file share.

## Deploying the application:
1. Deploying the backend:
  * Login to Azure: az login
  * Login to Azure Container Registry: az acr login --name aiassignmentdevcr 
  * cd src/ 
  * docker build -t aibackend:latest .
  * docker tag aibackend:latest aiassignmentdevcr.azurecr.io/aibackend:latest
  * docker push aiassignmentdevcr.azurecr.io/aibackend:latest
2. Deploying the frontend
  * First time you may need to initialize the project: swa init
  * cd ui-platform/
  * swa build
  * swa deploy --deployment-token=<deployment_token> -env=prod

- Note: you can authenticate with username=admin, password=Test!234
- Note: you can get the deployment_token from the env file
- Note: to deploy the preview mode, simply skip the -env param

## Create Sample Assignment Definitions

1. Open src/python/components/api/cli/create-assignment-definition.py and adjust the following variables:
  * assignment_definition_data
  * parts_data

2. Run the script

```bash
cd src/python/components

python api/cli/create-assignment-definition.py
```

## Database integration with alembic

### Configuration
The database connection URL is dynamically loaded from the .env file using python-dotenv. Add the following line to your .env file:

``` SQLALCHEMY_URL=postgresql+psycopg2://<username>:<password>@<host>:<port>/<database>```

### Running Migrations

cd .\src\python\components

Check the Current Migration State
#### To verify the database migration state:

`alembic current`

#### Apply New Migrations
To run all pending migrations:


`alembic upgrade head`

#### Roll Back Migrations
To undo the last migration:

` alembic downgrade -1` 
#### Generate a New Migration
To create a new migration script after updating your models:

`alembic revision --autogenerate -m "Your migration message"`


## Running tests

### Prerequisite
Have a docker running on your machine. Rancher Desktop is the recommended tool for local development. Tested configuretion: moby virtual machine, without kubernetes installed.

Before running the tests, run the following command:

```shell
export DOCKER_HOST=unix://$HOME/.rd/docker.sock
```

### Make sure you installed all the dependencies:
```shell
pip install -r requirements.txt
```

### Setup PYTHONPATH
```shell
cd src/python/components
export PYTHONPATH=.
```

### Run the tests
```shell
pytest
```

### Notes
Factories have also been created for all the models, but I couldn't find a way to make them work just yet. So for now,
we will have to manually provide data as a dict.

We have to do this:

```python
API_PREFIX = '/api/v1'

def test_create_student(client):
    payload = {"student_name": "Alice Wonderland"}
    
    resp = client.post(f"{API_PREFIX}/students", json=payload)
    assert resp.status_code == 201
    
    data = resp.json()
    assert data["student_name"] == "Alice Wonderland"
```

Instead of this:
```python
from api.tests.factories import CustomBaseFactory, StudentFactory

def test_create_student(db_session):
    student = StudentFactory()
    assert student.student_name is not None
```

# **Moodle Integration **

This guide outlines the steps to set up a local Moodle instance and integrate it the grading system using FastAPI.

## **1. Start Moodle Locally**
1. Navigate to the Moodle project directory:
   ```bash
   cd moodle
   ```
2. Start Moodle using Docker Compose:
   ```bash
   docker-compose up 
   ```
3. Once the containers are running, go to the Moodle site in your browser and log in using the configured username and password.
     ```
    http://localhost:8080/

    ```

## **2. Enable Web Services**
1. Navigate to **Site Administration** > **Advanced Features**.
2. Enable **Web services** and **REST protocol**.

## **3. Create an External Service**
1. Go to **Site Administration** > **Plugins** > **Web Services** > **External Services**.
2. Click Upload feedback files**Add a new service** and name it accordingly.
3. Set the service to **Enabled**.

## **4. Enable Required Web Service Functions**
1. Navigate to the newly created service.
2. Enable the following functions:
   - `mod_assign_get_submissions` (Retrieve assignment submissions)
   - `mod_assign_save_grades` (Upload grades)
   - `core_files_upload` (Upload feedback files)
   - `core_course_get_courses` (Get courses)
   - `core_course_get_updates_since` (get updates since the given timestamp)
   - `core_files_get_unused_draft_itemid` (Up files)
   - `mod_assign_get_assignments` (Fetch assignments)
   - `mod_assign_get_submission_status` (Get submission status)

## **5. Generate and Assign a Token**
1. Go to **Site Administration** > **Plugins** > **Web Services** > **Manage Tokens**.
2. Click **Add a new token** and:
   - Assign it to your **newly created external service**.
   - Link it to a user with appropriate permissions.
   - Copy and save the generated token for API authentication.

## **6. Create a Course and Assignment**
1. Navigate to **Site Administration** > **Courses** > **Add a new course**.
2. Enable **Editing Mode** and create a new assignment.
3. In the assignment settings, enable:
   - **Feedback comments**
   - **Feedback files**

## **7. Create and Enroll a Student**
1. Create a new student user under **Site Administration** > **Users**.
2. Enroll the student in the course under **Participants**.

## **8. Submit an Assignment for the Student**
1. Log in as the student and navigate to the assignment.
2. Submit a test assignment to simulate the workflow.

## **9. Trigger Grading**
1. Start the backend application locally:
2. Use the `/api/v1/trigger-sync` endpoint to trigger the sync job manually
