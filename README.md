# Superset Dashboards for iDempiere

This repository contains a set of Apache Superset dashboards designed to connect directly to an iDempiere ERP PostgreSQL database. These dashboards provide actionable insights across Sales, Inventory, and a pivot table to analyze Sales History, leveraging the raw transactional data from iDempiere.

## Prerequisites

- iDempiere ERP running with PostgreSQL backend
- Apache Superset
- Docker and Docker Compose (recommended for testing)
- Access to iDempiere database (read-only user recommended)

## Installation

1. **Clone Superset Docker Setup**
   ```bash
   git clone --depth=1  https://github.com/apache/superset.git
2. **Start Superset via Docker Compose**
   ```bash
   cd superset
   SUPERSET_LOAD_EXAMPLES=no docker compose -f docker-compose-image-tag.yml up --detach
   docker logs -f superset_app
3. **Configure Postgres to accept connections from docker container**
    As root user, edit the file pg_hba.conf on your system and add the following lines:
    ```
    # IPv4 docker Superset docker
    # !!! WARNING!! just for development or tests, not for production sites
    host    all             all             172.0.0.0/8             scram-sha-256
    ```
    As root user, edit the file postgresql.conf on your system and change the following line (if needed):
    ```
    listen_addresses = '*'			# what IP address(es) to listen on;
    ```
    And then reload the postgresql:
    ```
    sudo systemctl reload postgresql
    ```
4. **Get the IP address of your docker gateway**
    Get the IP address of your docker gateway with the following instruction:
    ```docker network inspect bridge```
    Take note of the address marked as Gateway, it must be something like 172.17.0.1 or 172.18.0.1

5. **Configure an iDempiere Database Connection within Superset**
    In your browser navigate to http://127.0.0.1:8088
    Log into Superset as user admin, password admin
    Navigate to Settings > Database Connections
    Click the button ```+Database```
    Create a new connection with the following data:
    - **HOST** = ```172.17.0.1```     -> This is, the result of the Gateway you obtained above, if your PostgreSQL is in another server, juts fill here the IP address of that PostgreSQL server
    - **PORT**          = ```5432```    -> or the PostgreSQL port used in your server
    - **DATABASE NAME** = ```idempieredev```  -> set here the name of your iDempiere database
    - **USERNAME**      = ```adempiere```
    - **PASSWORD**      = ```adempierepwd```   -> The password of your adempiere user in the database (this will be asked again later)
    - **DISPLAY NAME**  = ```idempiere```     -> **Very important!**  If you set a different name the import will fail

    Check that your connection is working before proceeding with the import.
6. **Clone the idempiere-superset repository**
    ```bash
    cd ..
    git clone https://github.com/laydasalasc/idempiere-superset
7. **Configure your database connection on the import file**
    Edit the file ```idempiere-superset/iDempiere-Dashboards/databases/idempiere.yaml``` and change the line:
    ```sqlalchemy_uri: postgresql+psycopg2://adempiere:XXXXXXXXXX@172.17.0.1:5432/idempieredb```
    replacing the IP address, port and the database if needed to match the same database you created above.
8. **Create the Superset zip file to import**
    ```bash
    cd idempiere-superset
    zip -9 iDempiere-Dashboards.zip iDempiere-Dashboards
9. **Import Dashboards in Superset**
    Back to your browser with the Superset session, navigate to ```Dashboards``` and click the ```Import Dashboards button```.  This is the button at the end of the toolbar, below Settings.
    Click the ```Select File``` button and choose the iDempiere-Dashboards.zip file you created above, then click ```Import``` and Superset will ask for your database password.
    After you fill the passwords the dashboards will be imported and ready to use.

9. **Security**
    The dashboards imported don't have any tenant security, after imported you can create a Tenant Role assigning access to all datasources, then create a User and assign the Gamma role and the Tenant Role created.
    And then you can create a Row Level Security rule with the clause ```ad_client_id=11``` (this is GardenWorld, you must change it to your AD_Client_ID), assign this RLS Rule to all the datasets and the Gamma and Tenant Roles.

10. **Extending**
    Please fork and submit pull requests for new dashboards, optimizations, or bug fixes.
    Open issues for ideas of new dashboards or charts, the idea is to maintain here dashboards and charts working on iDempiere vanilla.
    Ensure queries are safe, performant and schema-agnostic.  All queries must have the column ad_client_id in the output (to keep the security of the RLS Rule).

License: GPLv2 or later

Maintainer:
    - Layda Salas
    - layda.salas@globalqss.com
    - https://github.com/laydasalasc
