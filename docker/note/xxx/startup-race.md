# Startup Race

A **startup race** (or **race condition during startup**) occurs when multiple services (in Docker Compose) start simultaneously, and one service tries to connect to another before that service is fully ready to accept connections.

**Example scenario:**

- Web service starts and immediately tries to connect to a database.

- The database container is still initializing (loading schema, starting the listener).
    
- The web service fails to connect and crashes before the database finishes starting.

Common symptoms:

- Services fail with "Connection refused" or "Cannot connect to host".

- Services exit with errors like `ECONNREFUSED` or `psql: could not connect to server`.

- Restarting containers manually makes everything work.

**Solutions:**

1. Use `depends_on` with `condition` (Compose v2.1+):

    ```docker
    services:
      web:
        depends_on:
          db:
            condition: service_healthy
      db:
        healthcheck:
          test: ["CMD", "pg_isready", "-U", "postgres"]
          interval: 5s
          timeout: 5s
          retries: 5
    ```

2. Retry logic in application: Web service retries connecting to the database with exponential backoff instead of failing immediately.

3. Wait scripts: Use a script (like `wait-for-it.sh`) that polls the dependent service until it's ready.

4. Increase startup time: Give services more time to initialize before dependent services attempt connection.

5. Health checks: Define health checks so Compose knows when services are truly ready.

The `condition: service_healthy` approach is the most reliable - it ensures the dependent service doesn't start until the dependency is actually healthy, not just running.