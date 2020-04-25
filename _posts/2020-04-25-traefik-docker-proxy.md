---
title: Zero-config web proxying for Docker containers using Traefik
---

Set up [Traefik](https://docs.traefik.io) to automatically proxy a web application running inside a Docker container to an arbitrary hostname (e.g. `echo.mydomain.com`):

1. Instantiate a Docker image, giving it a name matching your desired hostname:

   ```sh
   docker run -d --name echo hashicorp/http-echo -text="Hello, World!"
   ```

2. Run Traefik with a `defaultRule` for your parent domain:
   {% raw %}

   ```sh
   docker run -d -p 80:80 -v /var/run/docker.sock:/var/run/docker.sock:ro traefik:2 --providers.docker --providers.docker.defaultRule='Host(`{{ .Name }}.mydomain.com`)'
   ```

   {% endraw %}

3. Set up a DNS record mapping your FQDN to your server's IP address
4. Visit http://echo.mydomain.com
