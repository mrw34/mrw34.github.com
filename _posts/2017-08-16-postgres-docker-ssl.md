---
layout: post
title: Enabling SSL for PostgreSQL in Docker
---
This script demonstrates how to enable SSL mode for a PostgreSQL server run in a Docker container using the official image.

`psql` automatically attempts to connect securely and "SSL connection" should be printed if successful.

This script uses a self-signed certificate for demonstration purposes - if you wish to make a client connection using JDBC you'll need to use a URL of the form `jdbc:postgresql:///postgres?ssl=true&sslfactory=org.postgresql.ssl.NonValidatingFactory`.

{% gist c97bb03ea1054afb551886ffc8b63c3b %}
