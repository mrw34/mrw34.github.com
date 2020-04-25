---
title: Enabling SSL for PostgreSQL in Docker
---

This script demonstrates how to enable SSL mode for a PostgreSQL server. It uses Docker but the same approach is valid when running a standalone server.

It starts the server, pauses whilst it initialises, and then uses the `psql` client to check that a secure connection can be established. "SSL connection" should be printed if it successful.

The script uses a self-signed certificate for demonstration purposes so if you wish to make a client connection using JDBC you'll need to use a URL of the form `jdbc:postgresql:///postgres?ssl=true&sslfactory=org.postgresql.ssl.NonValidatingFactory`.

{% gist c97bb03ea1054afb551886ffc8b63c3b %}
