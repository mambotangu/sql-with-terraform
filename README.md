# Objectives

1. Create a Cloud SQL instance
2. Install the Cloud SQL Proxy
3. Test connectivity with MySQL client using Cloud Shell

# Cloud SQL
Cloud SQL is a fully managed database service that makes it easy to set up, maintain, manage, and administer your relational databases on Google Cloud. You can use Cloud SQL with either MySQL or PostgreSQL.

# Step-by-step guide

## Download necessay files

1. Create a directory and fetch the required Terraform scripts from the Cloud Storage bucket with:
   
    mkdir sql-with-terraform \
    cd sql-with-terraform \
    gsutil cp -r gs://spls/gsp234/gsp234.zip

2. Unzip the downloaded content.

    unzip gsp234.zip

3. Update variables.tf file

### Run terraform module

    terraform init
    terraform plan -out=tfplan
    terraform apply tfplan

# Cloud SQL Proxy

## What the proxy provides

The Cloud SQL Proxy provides secure access to your Cloud SQL Second Generation instances without having to allowlist IP addresses or configure SSL.

Accessing your Cloud SQL instance using the Cloud SQL Proxy offers these advantages:

- Secure connections: The proxy automatically encrypts traffic to and from the database using TLS 1.2 with a 128-bit AES cipher; SSL certificates are used to verify client and server identities.

- Easier connection management: The proxy handles authentication with Cloud SQL, removing the need to provide static IP addresses.

Note: You do not need to use the proxy or configure SSL to connect to Cloud SQL from App Engine standard or the flexible environment. These connections use a "built-in" proxy implementation automatically.

### How the Cloud SQL Proxy works

The Cloud SQL Proxy works by having a local client, called the proxy, running in the local environment. Your application communicates with the proxy with the standard protocol used by your database. The proxy uses a secure tunnel to communicate with its companion process running on the server.

# Installing the Cloud SQL Proxy

1. Download the proxy:

wget https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64 -O cloud_sql_proxy

2. Make the proxy executable:

    chmod +x cloud_sql_proxy

You can install the proxy anywhere in your environment. The location of the proxy binaries does not impact where it listens for data from your application.

## Proxy startup options
When you start the proxy, you provide it with the following sets of information:

- What Cloud SQL instances it should establish connections to
- Where it will listen for data coming from your application to be sent to Cloud SQL
- Where it will find the credentials it will use to authenticate your application to Cloud SQL
  
The proxy startup options you provide determine whether it will listen on a TCP port or on a Unix socket. If it is listening on a Unix socket, it creates the socket at the location you choose; usually, the /cloudsql/ directory. For TCP, the proxy listens on localhost by default.

## Test connection to the database

1. Start by running the Cloud SQL proxy for the Cloud SQL instance:
   
   export GOOGLE_PROJECT=$(gcloud config get-value project)

   MYSQL_DB_NAME=$(terraform output -json | jq -r '.instance_name.value')
MYSQL_CONN_NAME="${GOOGLE_PROJECT}:us-central1:${MYSQL_DB_NAME}"

2. Run the following command:

    ./cloud_sql_proxy -instances=${MYSQL_CONN_NAME}=tcp:3306

3. Navigate to sql-with-terraform directory:

    cd ~/sql-with-terraform

4. Get the generated password for MYSQL:

    echo MYSQL_PASSWORD=$(terraform output -json | jq -r '.generated_user_password.value')

5. Test the MySQL connection:

    mysql -udefault -p --host 127.0.0.1 default

6. When prompted, enter the value of MYSQL_PASSWORD, found in the output above, and press Enter.

7. You should successfully log into the MYSQL command line. Exit from MYSQL by typing Ctrl + d.

8. If you go back to the first Cloud Shell tab you'll see logs for the connections made to the Cloud SQL Proxy.
