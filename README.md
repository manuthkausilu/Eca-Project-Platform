# Platform Repository Analysis

`platform/` is a Maven parent (`Eca-Project-Platform`) that groups the infrastructure services used by the rest of the system.

## Modules

- `config-server`
  - Artifact: `Config-Server`
  - Dependencies include Spring Cloud Config Server and Actuator
  - Configured on `server.port: 9000`
  - Reads config from:
    - `https://github.com/manuthkausilu/Eca-Project-Configurations.git`
    - search paths: `platform`, `services`

- `service-registry`
  - Artifact: `Service-Registry`
  - Eureka Server + Config Client
  - `application.yaml` points to `http://config.platform:9000`
  - `application-dev.yaml` points to `http://localhost:9000`

- `api-gateway`
  - Artifact: `Api-Gateway`
  - Spring Cloud Gateway + Eureka Client + Config Client + Actuator
  - `application.yaml` points to `http://config.platform:9000`
  - `application-dev.yaml` points to `http://localhost:9000`

## Technology baseline

- Java `25`
- Spring Boot `4.0.3`
- Spring Cloud `2025.1.0`

## Runtime wiring

- `config-server` should start first so other platform services can fetch configuration.
- `service-registry` provides service discovery.
- `api-gateway` depends on configuration and service discovery for routing.

## Build

```bash
mvn -f platform/pom.xml clean package
```

## Run options

### Option 1: Run each service directly

```bash
mvn -f platform/config-server/pom.xml spring-boot:run
mvn -f platform/service-registry/pom.xml spring-boot:run
mvn -f platform/api-gateway/pom.xml spring-boot:run
```

### Option 2: Run packaged jars with PM2

`platform/ecosystem.config.js` defines these apps:

- `config-server`
- `service-registry`
- `api-gateway`

Example:

```bash
pm2 start platform/ecosystem.config.js
```

## Notes and risks

- Non-config-server ports are likely externalized in the config repository, so they are not fixed in this workspace alone.
- Hostname `config.platform` must resolve in the target environment (or use dev profile for localhost references where available).

