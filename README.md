# Github

## Project theme

A service for team development.

## Target audience

- Total number of users `65+ million`
- Daily active users `24 million`
- Number of repositories `>200 million`

## Project MVP

1. Git protocol support
2. Authorization
3. Git flow

## RPS

### load profile

- Load profile was taken from personal use case
- Average size of git repository is `~ 15 Mb`. Information was fetch by [Github search query](https://github.com/search?l=&p=1&q=size%3A%3E1+sort%3Dsize&ref=advsearch&type=Repositories)
- `k` is coefficient for converting rps for one person into a load for a daytime audience

```
k = 24 * 10^6 / (24 * 60 * 60) = 277,78 u/s
```

| Operation                               | Count | Time period |
| --------------------------------------- | ----- | ----------- |
| donwload changes from remote repository | 5     | day         |
| upload changes to remote repository     | 3     | day         |
| download repository                     | 5     | month       |
| upload issue                            | 3     | month       |
| download issue                          | 3     | day         |
| upload comment                          | 2     | day         |
| download comments                       | 2     | day         |
| upload pull request                     | 1     | day         |
| download pull requests                  | 2     | day         |
| create repository                       | 4     | month       |

### donwload changes from remote repository

| Step | Method | Request Content-Length |
| ---- | ------ | ---------------------- |
| 1    | GET    | 180                    |
| 2    | GET    | 180                    |
| 3    | POST   | 483                    |

```
Authorize request: 5 * k = 1 389 rps (144 Kb/s)
```

```
Get git meta info from server: 5 * k = 1 389 rps (144 Kb/s)
```

```
Download diff: 5 * k = 1 389 rps (655.12 Kb/s)
```

### upload changes to remote repository

| Step | Method | Response Content-Length |
| ---- | ------ | ----------------------- |
| 1    | GET    | 165                     |
| 2    | POST   | 648                     |
| 3    | POST   | 483                     |

```
Authorize request: 3 * k = 833.34 rps (134 Kb/s)
```

```
Upload git meta to server: 3 * k = 833.34 rps (527.35 Kb/s)
```

```
Upload diff: 3 * k = 833.34 rps (393.07 Kb/s)
```

### download repository

| Step | Method | Response Content-Length |
| ---- | ------ | ----------------------- |
| 1    | GET    | 165                     |
| 2    | POST   | 728                     |
| 3    | POST   | 15501508                |

```
Authorize request: 5 * k / month = 44.8 rps (4.33 Kb/s)
```

```
Get git meta info from server: 5 * k / month = 44.8 rps (32 Kb/s)
```

```
Download repository: 5 * k / month = 44.8 rps (665 Mb/s)
```

### upload issue

| Method | Response Content-Length |
| ------ | ----------------------- |
| GET    | 3298                    |

```
3 * k / month = 26.88 rps (86.58 Kb/s)
```

### download issue

| Method | Response Content-Length |
| ------ | ----------------------- |
| GET    | 245733                  |

```
3 * k / month = 1.36 rps (0.8 Mb/s)
```

### upload comment

| Method | Response Content-Length |
| ------ | ----------------------- |
| GET    | 214958                  |

```
2 * k = 556 rps (114 Mb/s)
```

### download comments

| Method | Response Content-Length |
| ------ | ----------------------- |
| GET    | 510914                  |

```
2 * k = 556 rps (271 Mb/s)
```

### upload pull request

| Method | Response Content-Length |
| ------ | ----------------------- |
| GET    | 3450                    |

```
1 * k = 278 rps (935.88 Kb/s)
```

### download pull requests

| Method | Response Content-Length |
| ------ | ----------------------- |
| GET    | 295888                  |

```
2 * k = 556 rps (156 Mb/s)
```

### create repository

| Method | Request Content-Length |
| ------ | ---------------------- |
| POST   | 6240                   |

```
4 * k / month = 35.84 (219 Kb/s)
```

| Command                | RPS        | Bandwidth   |
| ---------------------- | ---------- | ----------- |
| authorize request      | 2 845 rps  | 500 Kb/s    |
| download git meta      | 1 389 rps  | 1 Mb/s      |
| upload git meta        | 833.34 rps | 527.35 Kb/s |
| download diff          | 1 389 rps  | 655.12 Kb/s |
| upload diff            | 833.34 rps | 393.07 Kb/s |
| download repository    | 44.8 rps   | 665 Mb/s    |
| create repository      | 35.84 rps  | 219 Kb/s    |
| upload issue           | 26.88 rps  | 86,58 Kb/s  |
| download issue         | 1.36 rps   | 0.8 Mb/s    |
| upload comment         | 556 rps    | 114 Mb/s    |
| download comments      | 556 rps    | 271 Mb/s    |
| upload pull request    | 278 rps    | 935.88 Kb/s |
| download pull requests | 556 rps    | 156 Mb/s    |

Peak load `x2.5`

```
Total peak load: 23 480 rps
```
```
Total Bandwidth in peak load: 3 Gb/s
```

## Storage

### Logical scheme

![database scheme](/assets/logical_scheme.jpeg)

### Physical scheme

![physical scheme](/assets/physical_scheme.jpeg)

### Data capacity

| Scheme        | Value                                                   | Size        |
| ------------- | ------------------------------------------------------- | ----------- |
| sessions      | (4 + 64 + 8 + 8) bytes * (24 * 10^6) users              | 2 Gb        |
| users         | (8 + 32 + 32) bytes * (65 * 10^6) users                 | 4.7 Gb      |
| issues        | (8 + 8 + 4 + 256) bytes * rps * 3600 * 24 * 31 * 12     | 235 Gb/year |
| repositories  | (8 + 4 + 4 + 66) bytes * (200 * 10^6) users             | 16.4 Gb     |
| pull_requests | (8 + 4 + 4 + 4) bytes * (200 * 365 * 10^6) users        | 165 Gb/year |
| comments      | (8 + 8 + 8 + 128 + 4) bytes * rps * 3600 * 24 * 31 * 12 | 2.5 Tb/year |
| packfiles     | 15 Mb * (200 * 10^6) repositries                        | 3 Pb        |

### RDB storage capacity

| Database | Size        | RPS    | Bandwidth |
| -------- | ----------- | ------ | --------- |
| meta     | 187 Gb/year | 20 628 | 400 Mb/s  |
| issues   | 235 Gb/year | 70     | 2.5 Mb/s  |
| comments | 2.5 Tb/year | 2 780  | 963 Mb/s  |

- meta database was sharded by field user_id in scheme Users.
- issues database was sharded by field repository_id in scheme Issues
- comments database was sharded by field repository_id in scheme Comments

```
Expected load to one shard is 3 500 rps
```

### In-memory storage capacity

| Database | Size | RPS    | Bandwidth |
| -------- | ---- | ------ | --------- |
| sessions | 2 Gb | 17 783 | 1.2 Mb/s  |

- sessions was sharded by user_id

```
Expected load to one shard is 10 000 rps
```

### Object storage capacity

| Database  | Size | RPS | Bandwidth |
| --------- | ---- | --- | --------- |
| packfiles | 3 Pb | 113 | 665 Mb/s  |

- packfiles was shareded by repository_id in filepath

```
Expected load to one shard is 665 Mb/s
```

## Architecture

![architecture](/assets/architecture.jpeg)

- [Github scaling story](https://github.blog/2021-04-29-scaling-monorepo-maintenance/)

`GitUpdater` - Service is responsible for update users [packfiles](https://git-scm.com/book/en/v2/Git-Internals-Packfiles). Gateway encodes and decodes git archive and iteracts with 
GitFlow to update meta information about user repositories and `Gateway` to send information to user.
Server must process a large amount of information from users in short time, so a [language without GC](https://blog.discord.com/why-discord-is-switching-from-go-to-rust-a190bbca2b1f) was chosen. The main advantage of rust in comparison with other low-level languages ​​is modern tooling and mechanisms for safe working with memory.

`GitFlow` - Service interacts with RDB and updte user repository meta information. Go was chosen for its speed of development and its ability to split the server into microservices due to its extensive networking capabilities.

`Sessions` - Authorization service interact with session storage.

`Gateway` - Supports user interface and is a common bus for integration of all services.

| Service    | RPS        | Bandwidth |
| ---------- | ---------- | --------- |
| GitUpdater | 113 rps    | 665 Mb/s  |
| GitFlow    | 23 478 rps | 1.4 Gb/s  |
| Sessions   | 17 783 rps | 1.2 Mb/s  |
| Gateway    | 23 480 rps | 3 Gb/s    |

### Users

Frontend are serving in CDN to reduce latency. For domain resolving will be picked high availability DNS service such as Route53 by Amazon. To reduce load and potential DDOS attack users traffic sanitize by web application firewall `cloudflare`. This enables set security configuration.

### Balancing

User requests are balanced between nginx servers using `DNS round-robin` algorithm.

### Application

One of the orchestration frameworks has been adopted to support policies such as automatic invocation, deployment, and support for various deployment types.
For enhancement security and infrastructure consistency level was used IaC and immutable infrastructure strategy.
To route traffic to the cluster, `istio` was adopted, which can manage not only all incoming external traffic, but also control all traffic within the cluster. Under the hood, `istio` uses Envoy as an additional proxy for each service. The observability layer consists of `prometheus operator`. For flexible routing and fault tolerance, a service mesh `linkerd` is used.

### Storage

Information are stored in HA clusters. Redis uses `cluster` and Postgres `patroni` to replicating data. Traffic is routed through TCP proxy. To eliminate single point of failure TCP proxy has two stand-by instances with keepalived connection.

### Availability zones

![regions](/assets/regions.jpeg)

| Regions      | AZ                               |
| ------------ | -------------------------------- |
| Americas     | Canada Central<br> East US       |
| Asia pacific | Australia East<br> Korea Central |
| Africa       | South Africa North               |
| Europe       | Germany West Central             |

## Hardware

![hardware](/assets/hardware.jpeg)

### MinIO cluster

- [Intel SaaS MinIO cluster overview](https://min.io/resources/docs/CPG-MinIO-implementation-guide.pdf)
- Server [supermicro](https://www.supermicro.com/en/products/MicroBlade/system/MBS-314E-6119M.cfm)
- Server blade module [supermicro](https://www.supermicro.com/en/products/MicroBlade/module/MBI-6119M-T2N.cfm)
- Disks [micron 8tb 2.5 NVMe](https://www.micron.com/products/ssd/product-lines/9200/9200-part-catalog?Value=8TB&Id={3125EB4D-9475-4DA9-B884-A31DF1CBA8DF})
