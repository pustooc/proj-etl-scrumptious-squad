# Data engineering end-to-end pipeline project

This project is a custom ETL pipeline. Data starts in an Amazon RDS postgres database, and is extracted to Parquet files stored in S3, transformed, and loaded into a postgres OLAP data warehouse. Data is therefore prepared for further work by Data Analysts.

<br />

## When setting up the project for the first time:

<br />

1. Setup the dev environment:
    1. Install WSL and VS Code. In the Linux terminal, type `code .` to shadow-install VS Code server.
    2. In Linux, install/update apt-get, git, psql, make, pyenv, python (`.python-version`), aws cli, jq, autopep8, and Terraform.

<br />

2. Deploy into a new AWS sandbox
    1. Create your AWS instance using the console
    2. Set up your AWS credentials locally by running
    ```
    aws configure
    ```
    3. In `terraform/`, change the region in `backend.tf` and `provider.tf` to match your instance's region
    4. In `terraform/create_secrets_bucket/s3.tf` rename `secrets_bucket` to your desired name for a secret bucket
    5. In `terraform/backend.tf` rename `backend s3`'s bucket to match the previous step
    6. In `terraform/vars.tf`, ensure the email within the email variable is correct
    7. From `project root`, run:
    ```sh
    cd terraform/create_secrets_bucket
    terraform init && terraform apply -auto-approve
    cd ..
    terraform init && terraform apply -auto-approve
    ```
    8. Verify that the SNS subscription is successful using the inbox of the email specified in step 3.6

<br />

3. Create .env files to store your credentials in a hidden way. Then upload them to AWS SecretsManager. This is required for the actual apps and the tests to work.
    1. For test credentials, run the following from `project root`, replacing the capitalised variables with your credentials:
    ```sh
    mkdir -p config
    touch config/.env.test
    echo 'database=SEED_TOTESYS_DATABASE_NAME' >> config/.env.test
    echo 'user=YOUR_LOCAL_POSTGRES_USERNAME' >> config/.env.test
    echo 'password=YOUR_LOCAL_POSTGRES_PASSWORD' >> config/.env.test
    ```
    2. For source Totesys database credentials, run the following from `project root`, replacing the capitalised variables with your credentials:
    ```sh
    mkdir -p config
    touch config/.env.development
    echo 'database=ACTUAL_TOTESYS_DATABASE_NAME' >> config/.env.development
    echo 'user=TOTESYS_USERNAME' >> config/.env.development
    echo 'password=TOTESYS_PASSWORD' >> config/.env.development
    echo 'host=TOTESYS_HOST' >> config/.env.development
    echo 'port=TOTESYS_PORT' >> config/.env.development
    ```
    3. For target Northcoders database credentials, run the following from `project root`, replacing the capitalised variables with your credentials:
    ```sh
    mkdir -p config
    touch config/.env.warehouse
    echo 'database=ACTUAL_NORTHCODERS_DATABASE_NAME' >> config/.env.warehouse
    echo 'user=NORTHCODERS_WAREHOUSE_USERNAME' >> config/.env.warehouse
    echo 'password=NORTHCODERS_WAREHOUSE_PASSWORD' >> config/.env.warehouse
    echo 'host=NORTHCODERS_WAREHOUSE_HOST' >> config/.env.warehouse
    echo 'port=NORTHCODERS_WAREHOUSE_PORT' >> config/.env.warehouse
    ```
    4. Upload the development credentials you just stored to AWS SecretsManager by running this script:
    ```sh
    source venv/bin/activate
    export PYTHONPATH=$(pwd)
    python src/set_up/run_make_secrets.py
    ```

<br />

4. Setup the seed Totesys database and Northcoders warehouse by running these two scripts from `project root`:
```sh
./extraction_test_db/run_seed_ext_test_db.sh
./load_test_warehouse_setup/run_seed_load_test_warehouse.sh
```

<br />

5. Check code safety and coverage and run tests with:
```sh
make run-checks
```

<br />

6. In your GitHub repo's secrets, make two new secrets. This is to enable CI/CD via GitHub Actions
    1. Name the first `AWS_ACCESS_KEY`, and in its contents copy-paste your AWS access key
    2. Name the second `AWS_SECRET_KEY`, and in its contents copy-paste your AWS secret key

<br />

## When working with the repo:

<br />

- Every time you come back to work on the repo (e.g. you've closed your IDE), activate your virtual environment (set up in step 3), set the correct path for importing packages and ensure PostgreSQL is started on your computer with:
```sh
source venv/bin/activate
export PYTHONPATH=$(pwd)
sudo service postgresql start
```

- When you've finished working, deactivate your virtual environment:
```sh
deactivate
```

<br />

## When your AWS instance is destroyed and you're remaking a new one:

<br />

- Follow step 2 again, but right before step 2.7, in both `terraform/create_secrets_bucket` and `terraform/`,
delete any folders and files beginning with `.terraform` and any `terraform.tfstate` files.
- Retry this step once more if step 2.7 fails.
