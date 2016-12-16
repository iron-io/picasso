# Picasso: Functions-as-a-Service (FaaS) on OpenStack

## Mission

Picasso aims to provide an API abstraction layer for Functions-as-a-Service (FaaS) on OpenStack.

## What is Serverless/FaaS?

Serverless is a new paradigm in computing that enables simplicity, efficiency and scalability for both developers
and operators. It's important to distinguish the two, because the benefits differ:

### Benefits for developers

The main benefits that most people refer to are on the developer side and they include:

* No servers to manage (serverless) -- you just upload your code and the platform deals with the infrastructure
* Super simple coding -- no more monoliths! Just simple little bits of code
* Pay by the milliseconds your code is executing -- unlike a typical application that runs 24/7, and you're paying
  24/7, functions only run when needed

Since you'll be running IronFunctions yourself, the paying part may not apply, but it does apply to
cost savings on your infrastructure bills as you'll read below.

### Benefits for operators

If you will be operating IronFunctions (the person who has to manage the servers behind the serverless),
then the benefits are different, but related.

* Extremely efficient use of resources
  * Unlike an app/API/microservice that consumes resources 24/7 whether they
    are in use or not, functions are time sliced across your infrastructure and only consume resources while they are
    actually doing something
* Easy to manage and scale
  * Single system for code written in any language or any technology
  * Single system to monitor
  * Scaling is the same for all functions, you don't scale each app independently
  * Scaling is simply adding more IronFunctions nodes


## System requirements

* Operating system: Linux/MacOS
* Python version: 3.5 or greater
* Database: MySQL 5.7 or greater

## Quick-start guide

* Install DevStack with [IronFunctions enabled](https://github.com/iron-io/picasso/blob/master/devstack/README.rst)
* Clone the [Picasso source](https://github.com/iron-io/picasso)

Create a Python3.5 virtualenv

    $ virtualenv -p python3.5 .venv
    $ source .venv/bin/activate

Install Picasso dependencies

    $ pip install -r requirements.txt -r test-requirements.txt

Install Picasso

    $ pip install -e .

Install MySQL if you haven't already, and create a new database for functions

    $ mysql -uroot -p -e "CREATE DATABASE functions"

### Migrations

Once all dependencies are installed it is necessary to run database migrations. First,
set the following environment variable:

    export PICASSO_MIGRATIONS_DB=mysql+pymysql://root:root@localhost/functions

Then use `alembic` to apply the migrations

    $ alembic upgrade head

### Starting the Picasso API server

    $ picasso-api --help

    Usage: picasso-api [OPTIONS]

      Starts Picasso API service

    Options:
      --host TEXT                    API service bind host.
      --port INTEGER                 API service bind port.
      --db-uri TEXT                  Picasso persistence storage URI.
      --keystone-endpoint TEXT       OpenStack Identity service endpoint.
      --functions-url TEXT           IronFunctions API URL
      --log-level TEXT               Logging file
      --log-file TEXT                Log file path
      --help                         Show this message and exit.

The following are the minimum required options to start the Picasso API service:

     --db-uri mysql://root:root@192.168.0.112/functions
     --keystone-endpoint http://192.168.0.112:5000/v3
     --functions-url http://192.168.0.112:8080/v1
     --log-level INFO

### Building and Running Picasso in Docker

Install [Docker engine](https://docs.docker.com/engine/installation/) if you haven't already.

From the Picasso repo, build a Docker image using the following commands:

    export DOCKER_HOST=tcp://<docker-host>:<docker-port>
    docker build -t picasso-api -f Dockerfile .

To start the container, pass in the required env vars with `--env-file` or by entering all required
values in `-e <KEY>=<VALUE>` format to the `docker run` command.

Example [Dockerfile.env](Dockerfile.env.example)

    docker run -d -p 10001:10001 --env-file Dockerfile.env picasso-api

Once the container is started, check if the service in running. In your web browser navigate to:

    <docker-host>:10001/api

or using the CLI:

    curl -X GET http://<docker-host>:10001/api/swagger.json | python -mjson.tool

### Examining the API

In [examples](examples/) folder you can find a script that examines available API endpoints.

Note that this script depends on the following env vars:

* `PICASSO_API_URL` - Picasso API endpoint
* `OS_AUTH_URL` - OpenStack Auth URL
* `OS_PROJECT_ID` - it can be found in OpenStack Dashboard or in CLI
* `OS_USERNAME` - OpenStack project-aligned username
* `OS_PASSWORD` - OpenStack project-aligned user password
* `OS_DOMAIN` - OpenStack project domain name
* `OS_PROJECT_NAME` - OpenStack project name

To run the script:

    OS_AUTH_URL=http://192.168.0.112:5000/v3 OS_PROJECT_ID=8fb76785313a4500ac5367eb44a31677 OS_USERNAME=admin OS_PASSWORD=root OS_DOMAIN=default OS_PROJECT_NAME=admin ./examples/hello-lambda.sh

Please note that values provided are project-specific, so they can't be reused.

### API docs

API docs are discoverable via Swagger. Just launch the Picasso API and browse to:

    http://<picasso-host>:<picasso-port>/api

### Support

Join us on [Slack](https://open-iron.herokuapp.com/)!