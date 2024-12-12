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
![Main_Archtecture](archtecture/Chromamon%20Data%20Flow.png)
 * **Data Ingestion**:
    + **Responsability**: Read data from a excel [file](/templates/Analysis%20spreadsheet%20template.xlsx) or a JSON object and send the data to a RabbitMQ queue.
    + **Technologies**: 
        + **Python**: Using **pandas** for the Excel file reading and **requests** to read JSON objects through a RestAPI endpoint.
        + **RabbitMQ**: Send the chromatography data to the analysis services.
        ![Ingestion](archtecture/Data%20Ingestion%20Data%20Flow.png)

* **Data Analysis**:
    + **Responsability**: Process the chromatographic data stored on the MongoDB Database and apply the diagnostics models (Rogers, IEEE, Duval and Key-Gas Analysis, or KGA).
    + **Technologies**:
        + **Java with Spring Batch**: For asynchronous processing.
        + **RabbitMQ**: To send the analysis results to the processor.
        ![Analysis](archtecture/Data%20Analysis%20Data%20Flow.png)

* **Data Processor**:
    + **Responsability**: Process the data from the analysis and enrich the overall data for the report service.
    + **Techonologies**:
        + **Spring Boot**: Along with JPA/Hibernate and drivers for MongoDB and PostgreSQL.
        + **PostgreSQL**: Store the information about the transformer.
        + **MongoDB**: Persist the processed and enriched data and store request information.
        ![Processor](archtecture/Data%20Processing%20Data%20Flow.png)

* **Reporting**:
    + **Responsability**: Generate reports in Word with the diagnosis and store the date of the next analysis.
    + **Technologies**: 
        + **Python**: Using **python-docx** library to write the documents.
        + **MongoDB**: Retrieve the data from the Processor.
        ![Report](archtecture/Data%20Report%20Data%20Flow.png)

* **Orquestration and Infrastructure**:
    + **Responsability**: Coordinate the execution of services, ensure communication between them and manage containers.
    + **Technologies**: 
        + **Docker and Kubernetes**: for containerization and orchestration.
        + **GitHub Actions**: for CI/CD.


## Data Flow

* **Data Ingestion**:
    + The Python service reads data from an Excel file (or receives a JSON via API).
    + Data is sent to RabbitMQ.
* **Analysis**:
    + A trigger in RabbitMQ notifies the Java (Spring Batch) service to start processing.
    + Data is processed and diagnostic models are applied.
    + The results are sent to RabbitMQ in another queue.
* **Processor**:
    + Information about transformers can be managed via API, with persistence in PostgreSQL.
* **Report**:
    + The Python service generates a Word report based on MongoDB results.

## Archtecture Components:

* **Clients**:
    + Operators use Excel files or REST APIs to provide data.

* **Microservices**:
    + **chroma-data-ingestion**: Processes Excel files and JSON Objects and sends data to RabbitMQ.
    + **chroma-duval-analysis**: Processes data using the Duval diagnostic models.
    + **chroma-rogers-analysis**: Processes data using the Rogers diagnostic models.
    + **chroma-ieee-analysis**: Processes data using the IEEE diagnostic models.
    + **chroma-kga-analysis**: Processes data using the Key-Gas Analysis diagnostic models.
    + **chroma-data-processing**: Manages transformer data.
    + **chroma-report-service**: Generates reports in Word and updates information of upcoming analyses.

* **Databases**:

    + **MongoDB**: Stores request data between the report service and the data processing service.
    + **PostgreSQL**: Schema: **report**
        + **Table: Transformer Data**: All the data related to the transformer.
        + **Table: Transformer Analysis**: Store all the chromatography data from previous analysis.
        + **Table: Transformer Result**: Store all the results from previous analysis.

* **Orchestration**:

    + **Kubernetes** to manage services.
    + **CI/CD with GitHub Actions** for build, test and deploy.