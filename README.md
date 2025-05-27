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

> In this exercise you will learn how to run ArangoDB using Docker, interact with its HTTP API through the command line, and preserve data using volumes. Furthermore, we will explore how ArangoDB's unique multi-model capabilities can be accessed in a containerized environment.

### Introduction

ArangoDB is a multi-model database that unifies document, key-value, and graph storage in a single engine using AQL (Arango Query Language). It provides an efficient way to handle complex, heterogeneous data without the overhead of managing multiple databases. Using Docker, you can easily deploy ArangoDB in a self-contained environment, experiment with its features, and maintain data persistence through volume mounts.

In this exercise, you will launch ArangoDB containers and interact with the system entirely through the command line using the `arangosh` shell and HTTP API. We'll explore creating databases and collections, inserting documents, querying data with AQL, and confirming persistence by comparing container instances with and without volume mounts. All interactions will use CLI tools to avoid reliance on any graphical interface.

### Further Readings and Other Sources

* [ArangoDB Docker Documentation](https://www.arangodb.com/docs/stable/deploy-docker.html)
* [ArangoDB AQL Documentation](https://www.arangodb.com/docs/stable/aql/)
* [Docker Volumes](https://docs.docker.com/storage/volumes/)

### Tasks

1. **Pull the ArangoDB Image:**

   ```bash
   docker pull arangodb
   docker images | grep arangodb
   ```

2. **Run ArangoDB Without Persistent Storage:**

   ```bash
   docker run --name arango-temp -e ARANGO_ROOT_PASSWORD=secret -d arangodb
   docker exec -it arango-temp arangosh --server.endpoint tcp://127.0.0.1:8529 \
     --server.password secret --javascript.execute-string \
     'db._createDatabase("testdb");'
   docker exec -it arango-temp arangosh --server.endpoint tcp://127.0.0.1:8529 \
     --server.username root --server.password secret --server.database testdb --javascript.execute-string \
     'db._create("users"); db.users.insert([{ _key: "u1", name: "Anna" }, { _key: "u2", name: "Ben" }, { _key: "u3", name: "Cara" }]);'
   ```

3. **Query the Documents via CLI:**

   ```bash
   docker exec -it arango-temp arangosh --server.endpoint tcp://127.0.0.1:8529 \
     --server.username root --server.password secret --server.database testdb --javascript.execute-string \
     'db._query("FOR user IN users FILTER user.name LIKE \"A%\" RETURN user").toArray();'
   ```

4. **Stop and Remove the Container:**

   ```bash
   docker stop arango-temp
   docker rm arango-temp
   ```

5. **Run ArangoDB With a Volume:**

   ```bash
   docker volume create arango-data
   docker run --name arango-persistent -e ARANGO_ROOT_PASSWORD=secret -d \
     -v arango-data:/var/lib/arangodb3 arangodb
   docker exec -it arango-persistent arangosh --server.endpoint tcp://127.0.0.1:8529 \
     --server.username root --server.password secret --server.database testdb --javascript.execute-string \
     'db._query("FOR user IN users RETURN user").toArray();'
   ```

6. **Verify Persistence:**

   ```bash
   docker stop arango-persistent
   docker rm arango-persistent
   docker run --name arango-persistent -e ARANGO_ROOT_PASSWORD=secret -d \
     -v arango-data:/var/lib/arangodb3 arangodb
   docker exec -it arango-persistent arangosh --server.endpoint tcp://127.0.0.1:8529 \
     --server.username root --server.password secret --server.database testdb --javascript.execute-string \
     'db._query("FOR user IN users RETURN user").toArray();'
   ```

### Questions

1. What makes ArangoDB a multi-model database, and how is it demonstrated in this exercise?
2. How can you confirm data persistence in ArangoDB using only the CLI?
3. What is the role of the `--javascript.execute-string` parameter in `arangosh`?
4. How would you back up and restore a Docker volume used by ArangoDB?
5. Can you adapt this setup for use with Docker Compose?

### Advice

Mastering ArangoDB through the command line not only boosts your confidence with shell tools but also prepares you for headless deployments in production environments. The combination of document and graph capabilities in a single tool gives you a strategic advantage in modeling flexible data structures. Practice these workflows regularly and try integrating them with container orchestration systems or backup tools to extend your understanding of persistent database deployments in modern DevOps pipelines.
