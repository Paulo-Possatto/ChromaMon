```
 ██████ ██   ██ ██████   ██████  ███    ███  █████  ███    ███  ██████  ███    ██ 
██      ██   ██ ██   ██ ██    ██ ████  ████ ██   ██ ████  ████ ██    ██ ████   ██ 
██      ███████ ██████  ██    ██ ██ ████ ██ ███████ ██ ████ ██ ██    ██ ██ ██  ██ 
██      ██   ██ ██   ██ ██    ██ ██  ██  ██ ██   ██ ██  ██  ██ ██    ██ ██  ██ ██ 
 ██████ ██   ██ ██   ██  ██████  ██      ██ ██   ██ ██      ██  ██████  ██   ████ 
```

# Introduction
ChromaMon (Chromatograhic Monitoring) is a service that reads transformers chromatographic analysis and generate a result of the condition of many transformers.

## Archtecture
ChromaMon has the following archtecture:
 * **Data Ingestion**:
    + **Responsability**: Read data from a excel [file](/templates/Analysis%20spreadsheet%20template.xlsx) or a JSON object and send the data to a MongoDB database for temporary storage.
    + **Technologies**: 
        + **Python**: Using **pandas** for the Excel file reading and **requests** to read JSON objects through a RestAPI endpoint.
        + **MongoDB**: Act as a buffer to retain the transformer data temporarilly.

* **Data Processing**:
    + **Responsability**: Process the chromatographic data stored on the MongoDB Database and apply the diagnostics models (Rogers, IEEE, Duval).
    + **Technologies**:
        + **Java with Spring Batch**: For asynchronous processing.
        + **PostgreSQL**: To store results and history.

* **Diagnostic Engine**:
    + **Responsability**: Apply diagnostic models and combine the results using the model priority strategy.
    + **Techonologies**:
        + **Java**: with logic based on the models.

* **Reporting**:
    + **Responsability**: Generate reports in Word with the diagnosis and store the date of the next analysis.
    + **Technologies**: 
        + **Python**: Using **python-docx** library to write the documents.
        + **MongoDB and PostgreSQL**.

* **Asset Management**:
    + **Responsability**: Allow CRUD of transformer information (inclusion, editing, deactivation).
    + **Technologies**:
        + **Java with SpringBoot**: Along with Hibernate/JPA dependencies.
        + **PostgreSQL**: for persistence.

* **Orquestration and Infrastructure**:
    + **Responsability**: Coordinate the execution of services, ensure communication between them and manage containers.
    + **Technologies**: 
        + **Docker and Docker Compose**: for containerization and orchestration.
        + **GitHub Actions**: for CI/CD.


## Data Flow

* **Data Ingestion**:
    + The Python service reads data from an Excel file (or receives a JSON via API).
    + Data is sent to MongoDB.
* **Processing & Diagnosis**:
    + A trigger in MongoDB notifies the Java (Spring Batch) service to start processing.
    + Data is processed and diagnostic models are applied.
    + The results are stored in PostgreSQL.
* **Report and Next Analysis**:
    + The Python service generates a Word report based on PostgreSQL results.
    + MongoDB is updated with the date of the next analysis and observations.
* **Transformer Management**:
    + Information about transformers can be managed via API, with persistence in PostgreSQL.

## Archtecture Components:

* **Clients**:
    + Operators use Excel files or REST APIs to provide data.

* **Microservices**:
    + **chroma-data-ingestion**: Processes Excel files and sends data to MongoDB.
    + **chroma-analysis-service**: Processes data using diagnostic models.
    + **chroma-transformer-management**: Manages transformer data.
    + **chroma-report-service**: Generates reports in Word and updates information of upcoming analyses.

* **Databases**:

    + **MongoDB**: Stores temporary data (such as recent analysis and upcoming executions).
    + **PostgreSQL**:
        + **Schema 1**: Transformer information.
        + **Schema 2**: Results of the analyses.

* **Orchestration**:

    + **Docker Compose** to manage services.
    + **CI/CD with GitHub Actions** for build, test and deploy.