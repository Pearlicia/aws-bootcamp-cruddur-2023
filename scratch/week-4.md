### Create rds postgres db via aws clickops

aws rds create-db-instance \
  --db-instance-identifier cruddur-db-instance \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version  14.6 \
  --master-username root \
  --master-user-password huEE33z2Qvl383 \
  --allocated-storage 20 \
  --availability-zone us-east-1a \
  --backup-retention-period 0 \
  --port 5432 \
  --no-multi-az \
  --db-name cruddur \
  --storage-type gp2 \
  --publicly-accessible \
  --storage-encrypted \
  --enable-performance-insights \
  --performance-insights-retention-period 7 \
  --no-deletion-protection

  ### Add time zone and character set

  ### After db creation connect to it from terminal using psql

  psql -U postgres --host localhost

  ### --host because of docker when not using docker no need for --host or -h

  ### Common PSQL commands:

\x on -- expanded display when looking at data
\q -- Quit PSQL
\l -- List all databases
\c database_name -- Connect to a specific database
\dt -- List all tables in the current database
\d table_name -- Describe a specific table
\du -- List all users and their roles
\dn -- List all schemas in the current database
CREATE DATABASE database_name; -- Create a new database
DROP DATABASE database_name; -- Delete a database
CREATE TABLE table_name (column1 datatype1, column2 datatype2, ...); -- Create a new table
DROP TABLE table_name; -- Delete a table
SELECT column1, column2, ... FROM table_name WHERE condition; -- Select data from a table
INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...); -- Insert data into a table
UPDATE table_name SET column1 = value1, column2 = value2, ... WHERE condition; -- Update data in a table
DELETE FROM table_name WHERE condition; -- Delete data from a table


### We can use the createdb command to create our database:
createdb cruddur -h localhost -U postgres

### We can create the database within the PSQL client
CREATE database cruddur;

### Delete database
DROP database cruddur;

### Import Script
### We'll create a new SQL file called schema.sql and we'll place it in backend-flask/db

### The command to import:

psql cruddur < db/schema.sql -h localhost -U postgres

Add UUID Extension
We are going to have Postgres generate out UUIDs. We'll need to use an extension called:

CREATE EXTENSION "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

### Add this to schema.sql file
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

### Then run this in terminal - cd to backend dir

psql cruddur < db/schema.sql -h localhost -U postgres

Password is password

### Connection String 
postgresql://postgres:pssword@127.0.0.1:5433/cruddur

### So to connect
psql postgresql://postgres:pssword@127.0.0.1:5433/cruddur

### Add to gitpod env
export CONNECTION_URL="postgresql://postgres:pssword@127.0.0.1:5433/cruddur"
gp env CONNECTION_URL="postgresql://postgres:pssword@127.0.0.1:5433/cruddur"

### Then 
psql $CONNECTION_URL

### PROD DB
export PROD_CONNECTION_URL="postgresql://cruddurroot:pssword@awspostgresendpoint:5433/cruddur"
gp env PROD_CONNECTION_URL="postgresql://cruddurroot:pssword@awspostgresendpoint:5433/cruddur"

### Then 
psql $PROD_CONNECTION_URL

### Create bin dir in backend then create bash files
### To run the bash files

ls -l ./bin      # to check if files have permissions
chmod u+x bin/db-create    # Add permissions to execute do it on all bin files

chmod +x bin/db-create    # Do this instead

./bin/db-drop or source /bin/db-drop

### Sed command
run man sed
sed - stream editor for filtering and transforming text
Sed  is  a stream editor.  A stream editor is used to perform basic text transformations on an input stream
(a file or input from a pipeline).  While in some ways

### Sessions script to see the running db processes run it to kill the processes

### lib/db.py
Add psycopg3 driver for connection, could use sqlalchemy orm instead

Connect to RDS via Gitpod
In order to connect to the RDS instance we need to provide our Gitpod IP and whitelist for inbound traffic on port 5432.

GITPOD_IP=$(curl ifconfig.me)
We'll create an inbound rule for Postgres (5432) and provide the GITPOD ID.

We'll get the security group rule id so we can easily modify it in the future from the terminal here in Gitpod.

export DB_SG_ID="sg-0b725ebab7e25635e"
gp env DB_SG_ID="sg-0b725ebab7e25635e"
export DB_SG_RULE_ID="sgr-070061bba156cfa88"
gp env DB_SG_RULE_ID="sgr-070061bba156cfa88"
Whenever we need to update our security groups we can do this for access.

``` aws ec2 modify-security-group-rules \
    --group-id $DB_SG_ID \
    --security-group-rules "SecurityGroupRuleId=$DB_SG_RULE_ID,SecurityGroupRule={IpProtocol=tcp,FromPort=5432,ToPort=5432,CidrIpv4=$GITPOD_IP/32}"
https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-security-group-rules.html#examples

```

### Create a lamda funtion in console with this code

Setup Cognito post confirmation lambda
Create the handler function
Create lambda in same vpc as rds instance Python 3.8
Add a layer for psycopg2 with one of the below methods for development or production
ENV variables needed for the lambda environment.

PG_HOSTNAME='cruddur-db-instance.czz1cuvepklc.ca-central-1.rds.amazonaws.com'
PG_DATABASE='cruddur'
PG_USERNAME='root'
PG_PASSWORD='huEE33z2Qvl383'

The original function


```
import json
import psycopg2

def lambda_handler(event, context):
    user = event['request']['userAttributes']
    try:
        conn = psycopg2.connect(
            host=(os.getenv('PG_HOSTNAME')),
            database=(os.getenv('PG_DATABASE')),
            user=(os.getenv('PG_USERNAME')),
            password=(os.getenv('PG_SECRET'))
        )
        cur = conn.cursor()
        cur.execute("INSERT INTO users (display_name, handle, cognito_user_id) VALUES(%s, %s, %s)", (user['name'], user['email'], user['sub']))
        conn.commit() 

    except (Exception, psycopg2.DatabaseError) as error:
        print(error)
        
    finally:
        if conn is not None:
            cur.close()
            conn.close()
            print('Database connection closed.')

    return event

```
Function is edited in the aws/lamdas/cruddur-post-confirmation.py file

### Add this to lambda console env 

CONNECTION_URL="postgresql://cruddurroot:pssword@awspostgresendpoint:5433/cruddur"

### Development
https://github.com/AbhimanyuHK/aws-psycopg2

This is a custom compiled psycopg2 C library for Python. Due to AWS Lambda missing the required PostgreSQL libraries in the AMI image, we needed to compile psycopg2 with the PostgreSQL libpq.so library statically linked libpq library instead of the default dynamic link.

EASIEST METHOD

Some precompiled versions of this layer are available publicly on AWS freely to add to your function by ARN reference.

https://github.com/jetbridge/psycopg2-lambda-layer

#### Add this arn below to lambda console layers, create layers then add
Just go to Layers + in the function console and add a reference for your region
arn:aws:lambda:ca-central-1:898466741470:layer:psycopg2-py38:1

Alternatively you can create your own development layer by downloading the psycopg2-binary source files from https://pypi.org/project/psycopg2-binary/#files

Download the package for the lambda runtime environment: psycopg2_binary-2.9.5-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl

Extract to a folder, then zip up that folder and upload as a new lambda layer to your AWS account

### Production
Follow the instructions on https://github.com/AbhimanyuHK/aws-psycopg2 to compile your own layer from postgres source libraries for the desired version.

Add the function to Cognito
Under the user pool properties add the function as a Post Confirmation lambda trigger.


#### Go to cognito and lambda triggers then add a trigger
Choose signup trigger
choose post confirmation trigger
choose lambda function - post confirmation 2

# Go watch the lambda video again for the VPC permission creation








  
