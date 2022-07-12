# TechConf Registration Website

## Project Overview
The TechConf website allows attendees to register for an upcoming conference. Administrators can also view the list of attendees and notify all attendees via a personalized email message.

The application is currently working but the following pain points have triggered the need for migration to Azure:
 - The web application is not scalable to handle user load at peak
 - When the admin sends out notifications, it's currently taking a long time because it's looping through all attendees, resulting in some HTTP timeout exceptions
 - The current architecture is not cost-effective 

In this project, you are tasked to do the following:
- Migrate and deploy the pre-existing web app to an Azure App Service
- Migrate a PostgreSQL database backup to an Azure Postgres database instance
- Refactor the notification logic to an Azure Function via a service bus queue message

## Dependencies

You will need to install the following locally:
- [Postgres](https://www.postgresql.org/download/)
- [Visual Studio Code](https://code.visualstudio.com/download)
- [Azure Function tools V3](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cbash#install-the-azure-functions-core-tools)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Azure Tools for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack)

## Project Instructions

### Part 1: Create Azure Resources and Deploy Web App
1. Create a Resource group
2. Create an Azure Postgres Database single server
   - Add a new database `techconfdb`
   - Allow all IPs to connect to database server
   - Restore the database with the backup located in the data folder
3. Create a Service Bus resource with a `notificationqueue` that will be used to communicate between the web and the function
   - Open the web folder and update the following in the `config.py` file
      - `POSTGRES_URL`
      - `POSTGRES_USER`
      - `POSTGRES_PW`
      - `POSTGRES_DB`
      - `SERVICE_BUS_CONNECTION_STRING`
4. Create App Service plan
5. Create a storage account
6. Deploy the web app

### Part 2: Create and Publish Azure Function
1. Create an Azure Function in the `function` folder that is triggered by the service bus queue created in Part 1.

      **Note**: Skeleton code has been provided in the **README** file located in the `function` folder. You will need to copy/paste this code into the `__init.py__` file in the `function` folder.
      - The Azure Function should do the following:
         - Process the message which is the `notification_id`
         - Query the database using `psycopg2` library for the given notification to retrieve the subject and message
         - Query the database to retrieve a list of attendees (**email** and **first name**)
         - Loop through each attendee and send a personalized subject message
         - After the notification, update the notification status with the total number of attendees notified
2. Publish the Azure Function

### Part 3: Refactor `routes.py`
1. Refactor the post logic in `web/app/routes.py -> notification()` using servicebus `queue_client`:
   - The notification method on POST should save the notification object and queue the notification id for the function to pick it up
2. Re-deploy the web app to publish changes

## Monthly Cost Analysis
Complete a month cost analysis of each Azure resource to give an estimate total cost using the table below:

| Azure Resource | Service Tier | Monthly Cost |
| ------------ | ------------ | ------------ |
| *Azure Database for PostgreSQL* | Single Server, Basic Tier, Gen 5, 1vCore, 5GB storage    |     25.32 USD      |
| *Azure Service Bus*   |    Basic     |      0.05 USD      |
| *Azure Web App*       |     F1: Free    |     0.00 USD   |
| *Azure Storage Account* |      StorageV2 Standard/Hot        |    0.04 USD    |
| *Azure Azure Function* |   Consumption Plan       |      0.00 USD    |
| **Total** |             |     25.41 USD      |

## Services Used

### Azure Database for PostgreSQL
Azure has a variety of managed services for databases. Azure Database for PostgreSQL is a relational database service based on the open-source Postgres database engine. It's a fully managed database-as-a-service that can handle mission-critical workloads with predictable performance, security, high availability, and dynamic scalability.

### Azure Storage Accounts
An Azure storage account can store data objects you create, such as blobs and files. This storage account provides a unique namespace in Azure for your data, and every item that you store has an address that includes your unique account name. I used the General-purpose v2 storage accounts, which provide support and the latest features for Azure Storage services such as blobs etc.

### Azure Functions
Azure Functions is a serverless, event-driven, compute-on-demand platform where we deploy cohesive functions (blocks of code) after building and debugging them locally. Azure functions are supported in both Linux and Windows environments and support many programming languages, such as C#, JavaScript, Java, and Python.

### Azure Web App
Azure App Service enables you to build and host web apps, mobile back ends, and RESTful APIs in the programming language of your choice without managing infrastructure. It offers auto-scaling and high availability, supports both Windows and Linux, and enables automated deployments from GitHub, Azure DevOps, or any Git repo. An Azure Web App is deployed within an App Service and can be deployed either as a codebase or a container.

### Azure Service Bus
Azure Service Bus is a fully managed enterprise message broker with message queues and publish-subscribe topics (in a namespace).

## Architecture Explanation

This is a small application that does not require 14 GB of RAM and more than 4 vCPUS and control over underlying system and configurations. Its scales easier with the app service than using lift and shift migration which is time consuming to configure.
Azure Function App is more convenient to use for the API because it, like App Service plan, also provides auto scaling capabilities and has built-in bindings to work with other Azure services.
