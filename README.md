# Voting App With Docker

A simple distributed application running across multiple Docker containers.

## Getting started

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/).

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

Run in this directory to build and run the app:

```shell
docker-compose up
```

The `vote` app will be running at [http://localhost:5000](http://localhost:5000), and the `results` will be at [http://localhost:5001](http://localhost:5001).

## Architecture

![Architecture diagram](architecture.excalidraw.png)

* A front-end web app in [Python](/vote) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) which collects new votes
* A [.NET](/worker/) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
* A [Node.js](/result) web app which shows the results of the voting in real time

## Usage

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

* **Processing flow:**
  * User submits vote via Voting App.
  * Redis temporarily stores data.
  * Worker processes and updates to PostgreSQL database.
  * Results App displays vote results.

**Deployment Images commands**:

```bash
# Voting App
docker build . -t voting-app
docker run -d --name=redis redis
docker run -d --name=voting_app -p 5000:80 --link redis:redis voting-app

# Worker App
docker build . -t worker-app
docker run -d --name=db -e POSTGRES_USER=<username> POSTGRES_PASSWORD=<password> postgres:15
docker run -d --name=worker --link redis:redis --link db:db worker-app

# Result App
docker build . -t result-app
docker run -d --name=result_app -p 5001:80 --link db:db result-app
```
