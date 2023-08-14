# grafana + prometheus + query-exporter

![screenshot](./images/screenshot.png)

Quick Start your RDMMS metrics visualization with grafana + prometheus + query-exporter!

## Overview

![architecture](./images/architecture.png)

The core components in the architecture are **Grafana** and **Prometheus**.

* **Prometheus** is responsible for collecting and storing metric data.
* **Grafana** acts as the frontend that visualizes data queried from Prometheus using PromQL.

The **query-exporter** executes SQL queries from your RDBMS, transforming the results into metrics. These metrics will be periodically collected by Prometheus. For instance, this could include metrics such as the number of subscribers or the count of posts.

To collect metrics from the RDBMS, please open the `query-exporter/config.yaml` file and configure the DB connection settings, as well as write the necessary SQL queries. Below is an example!

```yaml
databases:
  my-db:
    dsn: mysql://my-username:my-password@my-database-host:3306/my-database

metrics:
  my_user_total:
    type: gauge
    description: Total user count

queries:
  user-total-query:
    interval: 5
    databases: [my-db]
    metrics: [my_user_total]
    sql: SELECT COUNT(*) AS my_user_total FROM member
```

And run this command:
```bash
docker compose up
```
If you are using docker compose V1, then:
```bash
docker-compose up
```

Open the browser and head over to `http://localhost:3000`. Initial username and password is `admin` / `admin`.


## Bonus: Connect database over SSH Tunnel

If your database is located remotely and its port is not directly accessible, you might find it challenging to connect directly. In such cases, consider using an SSH Tunnel. An SSH Tunnel allows you to access a remote database through SSH.

Add the following content to your docker-compose.yml:
```yaml
version: "3"
services:
  # ...

  my-db:
    image: solo5star/ssh-tunnel
    restart: unless-stopped
    volumes:
      - ./ssh:/ssh
    environment:
      - TUNNEL_HOST=example-server.com
      - LOCAL_PORT=3306
      - REMOTE_HOST=localhost
      - REMOTE_PORT=3306
```

Create an `ssh/config` file and write the following content. The format is the same as the commonly known OpenSSH config. If necessary, place the PEM key inside the ssh folder.

```
HOST example-server.com
    HostName example-server.com
    Port 22
    User example-user
    IdentityFile ~/.ssh/example-server.pem
```

By doing so, you'll be able to connect to the database via the address "my-db" from the query-exporter!
