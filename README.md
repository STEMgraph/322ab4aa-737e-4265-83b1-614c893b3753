<!---
{
  "id": "322ab4aa-737e-4265-83b1-614c893b3753",
  "depends_on": ["d1bee1c7-d88a-4f00-a44e-3e402f6ee826"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-05-14",
  "keywords": ["ArangoDB", "Docker", "Multi-Model Database", "Persistent Storage", "Graph Database"]
}
--->

# Using ArangoDB in a Docker Container

> In this exercise you will learn how to run ArangoDB using Docker, interact with its HTTP API and Web UI, and preserve data using volumes. Furthermore, we will explore how ArangoDB's unique multi-model capabilities can be accessed in a containerized environment.

### Introduction

ArangoDB is a multi-model database that combines the features of document stores, key-value pairs, and graph databases in a single engine using AQL (Arango Query Language). It is ideal for use cases requiring flexibility between different data models. Deploying ArangoDB via Docker is a convenient and reproducible way to experiment with these capabilities in isolation.

In this exercise, we begin by launching an ArangoDB container without persistent storage to demonstrate data loss upon container deletion. You will access the Web UI, create a database and collections, and insert sample documents. After observing the transient nature of the data, we'll redeploy the container with a mounted volume to show how data can persist across sessions. This includes the re-creation and verification of inserted content after the container is removed and relaunched.

### Further Readings and Other Sources

* [ArangoDB Docker Documentation](https://www.arangodb.com/docs/stable/deploy-docker.html)
* [ArangoDB AQL Documentation](https://www.arangodb.com/docs/stable/aql/)
* [Docker Volumes](https://docs.docker.com/storage/volumes/)

### Tasks

1. **Pull the ArangoDB Image:**

   ```bash
   docker pull arangodb
   ```

   Confirm the image is available locally:

   ```bash
   docker images | grep arangodb
   ```

2. **Run ArangoDB without Persistent Storage:**

   ```bash
   docker run --name arango-temp -e ARANGO_ROOT_PASSWORD=secret -d -p 8529:8529 arangodb
   ```

   Visit `http://localhost:8529` and log in with:

   * **Username:** root
   * **Password:** secret

3. **Create a Database and Collection in the Web UI:**

   * Navigate to "Databases" > "+" > Name it `testdb`.
   * Enter the new database and create a new collection named `users`.
   * Insert 3 documents using the UI:

     ```json
     { "_key": "u1", "name": "Anna" }
     { "_key": "u2", "name": "Ben" }
     { "_key": "u3", "name": "Cara" }
     ```

4. **Query the Documents:**
   Use the "AQL Editor" to execute:

   ```aql
   FOR user IN users FILTER user.name LIKE "A%" RETURN user
   ```

5. **Stop and Remove the Container:**

   ```bash
   docker stop arango-temp
   docker rm arango-temp
   ```

6. **Run ArangoDB with a Volume:**

   ```bash
   docker volume create arango-data
   docker run --name arango-persistent -e ARANGO_ROOT_PASSWORD=secret -d -p 8529:8529 \
     -v arango-data:/var/lib/arangodb3 arangodb
   ```

   Repeat steps 3–4 to recreate the data.

7. **Verify Persistence:**

   ```bash
   docker stop arango-persistent
   docker rm arango-persistent
   docker run --name arango-persistent -e ARANGO_ROOT_PASSWORD=secret -d -p 8529:8529 \
     -v arango-data:/var/lib/arangodb3 arangodb
   ```

   Log back into the Web UI and check if the `testdb` and `users` collection with documents still exist.

### Questions

1. What makes ArangoDB a multi-model database, and how is it used here?
2. What port must be exposed to access the ArangoDB Web UI?
3. Why is persistent storage crucial for databases in Docker?
4. How would you back up the ArangoDB data volume?
5. Can you use Docker Compose for ArangoDB? If so, how?

### Advice

This exercise not only strengthens your understanding of Docker containers but also introduces you to ArangoDB's powerful capabilities in a reproducible environment. Embrace the multi-model approach — try graph or key-value queries next. Use this sheet as a foundation and continue exploring ArangoDB’s unique features like Foxx microservices and smart joins. Consider integrating ArangoDB into small apps to understand how a containerized multi-model DBMS can scale and support diverse data needs.
