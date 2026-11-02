# Repository inventory

Shallow inventory of the company repositories currently attached to this Cursor workspace. Produced 2026-11-02.

`icepanel-poc` is excluded. The parent `TravelStart` directory was not scanned. Inspection used READMEs, Gradle and npm manifests, Helm charts, Dockerfiles, and top-level directory layout. `node_modules`, `vendor`, `build`, `dist`, `.output`, `.nuxt`, `.gradle`, `coverage`, and similar generated trees were skipped. Secret files were not opened.

No repository was treated as a microservice unless its own deployment evidence says it is one independently deployable application. None of these five repositories is a shared library.

Labels: **confirmed** means a file in that repository states it. **inferred** means the layout suggests it and a second source was not checked. **unknown** means this pass did not establish it.

## Summary

| Repository | Languages | Application type | Independently deployable | Multiple modules | Infrastructure | API specification | Shared library | Needs investigation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `b2c-website-whitelabel-flights` | TypeScript, Vue, SCSS | Customer website with an in-process BFF | Yes | One application | Yes | Yes, checked in | No | Yes |
| `payments-service` | Java | Payments modular monolith | Yes | Several Gradle modules, one deployable | Yes | Runtime docs only | No | Yes |
| `travelstart-account` | Java, SQL, XML | Account HTTP API | Yes | Several Gradle modules, one deployable | Yes | Postman collection and code-configured Swagger | No | Yes |
| `ts-tcc` | Java, XHTML, JavaScript, CSS | Server-rendered internal web application | Appears so, as a Tomcat WAR | One application | No in-repo deploy config | None found | No | Yes |
| `ts-web-api` | Java, XML | JSON HTTP API for the flights website | Yes | One application | Yes | Runtime Swagger only | No | Yes |

## b2c-website-whitelabel-flights

Customer-facing multi-tenant flight booking website. Backstage catalog names the component `b2c-whitelabel-flights`, type `website`, lifecycle `experimental`, system `storefront` (`catalog-info.yaml`). One Nuxt application lives in `layers/core-base` and includes the browser app and the Nitro BFF in the same build (`layers/README.md`, `package.json`). `package.json` is `private: true`.

**Confirmed:** TypeScript, Vue, and SCSS are the main languages. Frameworks named by the manifest are Nuxt 4, Vue 3, Pinia, PrimeVue, Tailwind, Vue Router, and Sentry. Node 24. Own Helm chart `ts-web-whitelabel-flights` (`type: application`), `Dockerfile` (Node 24, runs `.output`), `docker-compose.yml`, `docker-compose.redis.yml`, and GitHub workflows under `.github/workflows/`. OpenAPI 3.1 document at `docs/openapi/bff.yaml`, titled "Travelstart White-Label BFF API".

**Inferred:** Independently deployable, because the chart and container package this repository alone. Not a shared library.

**Unknown:** Whether `tools/understand-anything` is product code or local tooling. It has its own `package.json` files and was not treated as a second application. The BFF document names Account and WAPI as upstreams; those calls were not traced in code.

## payments-service

Domain-agnostic payments processor. The README calls it a modular monolith for multiple payment service providers. Gradle root name `payments-service`. Modules from `settings.gradle.kts`: `:api`, `:core`, `:gateways:peach:checkout`, `:gateways:peach:s2s`.

**Confirmed:** Java only (322 `.java` files; no Kotlin sources). Java 21 (`build.gradle.kts`). README tech stack: Spring Boot 3.5.10, PostgreSQL 17, Gradle 9.2, Docker. `api/build.gradle.kts` applies the Spring Boot plugin and builds a Boot JAR. `Dockerfile` copies `api/build/libs/api.jar` and listens on 8082. Helm chart `ts-payments-service` (`type: application`), plus `docker-compose.yml` and `.github/workflows/`. Springdoc is a dependency of `:api` and `:core`. README points operators at `http://localhost:8082/swagger-ui.html`. Flyway migrations live under `core/src/main/resources/db/migration/`.

**Inferred:** The gateway modules are libraries inside the one service. They are not separate deployables: only `:api` applies the Spring Boot plugin, and the image runs a single JAR.

**Unknown:** No checked-in OpenAPI file was found. The live spec was not fetched. How callers reach this service was not traced.

## travelstart-account

Account HTTP API. README title is "Travelstart Account Api". Entry class `api/.../TravelstartAccountApiApplication.java`. Gradle root name `com.travelstart.account`. Included modules: `core`, `api`, `persistence`, `common`, `log`, `dispatchers`.

**Confirmed:** Java, with SQL and XML alongside it. README requires Java 8. Root `build.gradle` sets Spring Boot `1.5.12.RELEASE`. `:api` applies the Spring Boot plugin. `Dockerfile` copies `api/build/libs/*.jar` to `tsacct-api.jar` on a Java 8 JRE. Helm chart `ts-account-api` (`type: application`). Also present: `docker-compose.yml`, `Jenkinsfile`, `Jenkinsfile-vmware`, and `.github/workflows/beta.yaml`. `persistence` and `log` apply the Spring Boot plugin and set `bootRepackage.enabled = false`. API surface artifacts found: `postman/travelstart-account.postman_collection.json`, `api/.../SwaggerConfig.java`, and Springfox dependencies. README documents Swagger UI at `http://localhost:7000/swagger-ui.html` and Liquibase for the database.

**Inferred:** One deployable API. `common` and `core` are internal modules, not published libraries. `core` contains only `build.gradle`. `dispatchers` has no source, only a `build` directory.

**Unknown:** Which deployment path is current. The README describes AWS CodeDeploy blue-green, Jenkins job `ts-account-api-multi-pipeline`, and Chef templates in a separate `ts_chef` repository. This repository also contains a Helm chart and a container image. Deprecation is not stated. No standalone OpenAPI file was found.

## ts-tcc

Single Gradle project, root name `ts-tcc`. It builds `tcc-web.war` with context name `tcc` (`build.gradle`). Java packages sit under `com.travelstart.tcc`, including `web`, `bookingdisplay`, `invoice`, and `security`. The `web/` tree holds XHTML, JavaScript, CSS, and AdminLTE assets.

**Confirmed:** Java 8, Spring 4.3.11, Apache Camel 2.17.3, JSF 2.2 (`jsf-api`, `jsf-impl`) and Tomahawk, packaged with the Gradle `war` plugin. The same build file resolves `com.travelstart` artifacts `ts-api-client`, `ts-common`, `ota`, and `ts-web-common` from Nexus. Those artifacts are not repositories in this workspace. README setup is a local Apache Tomcat 9 deploy via `./gradlew deploy`.

**Inferred:** An internal server-rendered web application, from the WAR, JSF, and the `tcc` web packages. It consumes shared libraries. It is not itself a shared library.

**Unknown:** What "TCC" stands for. The README does not describe the product. No Dockerfile, Helm chart, Compose file, Jenkinsfile, or `.github` workflows were found, so production deployment is not evidenced in this repository. No API specification file was found. The source does not say the system is deprecated. The Java 8 / Spring 4 / JSF stack is older than `payments-service` and the Nuxt website.

## ts-web-api

JSON API used by the flights website. README: "This API is used by the flights Angular website to connect to things using JSON." Single Gradle project, root name `ts-web-api`. Builds `website-services.war` with context name `website-services` (`build.gradle`).

**Confirmed:** Java 8, Spring 4.3.24, Apache Camel 2.19.5, Gradle `war` plugin. `Dockerfile` builds that WAR on Temurin 8. Helm chart `ts-web-api` (`type: application`). GitHub workflows: `beta.yml`, `preprod.yml`, `prod.yaml`, `release.yaml`. `Jenkinsfile.not.used` is present. `platform-config/templates/systems` holds configuration templates. `SecureRestSwaggerServlet.java` exists, and the README documents Swagger at `/website-services/api-docs`. The build resolves `com.travelstart` artifacts from GitHub Packages (`travelstart/ts-commons`, `travelstart/api-client`) and optionally Nexus. Those repositories are not in this workspace.

**Inferred:** One independently deployable WAR, not a shared library. `platform-config` is configuration for this application, not a second application.

**Unknown:** No checked-in OpenAPI or Swagger file was found. The README's Angular client is not in this workspace, and this pass did not compare that client with `b2c-website-whitelabel-flights`. The filename `Jenkinsfile.not.used` conflicts with the Helm chart; which path deploys production was not verified. The source does not say the system is deprecated.

## Further investigation

These items are **unknown** until a later pass reads code or deployment config.

1. `ts-tcc` has no in-repo cluster or container deployment. Confirm where `tcc-web.war` runs, and what TCC means.
2. `travelstart-account` documents CodeDeploy and Chef, and also ships Helm and Docker. Confirm the live path. Confirm that `core` and `dispatchers` are unused.
3. `ts-web-api` has both a Helm chart and `Jenkinsfile.not.used`. Confirm the live path, and whether the Nuxt site, an Angular site, or both call it.
4. `payments-service` publishes OpenAPI at runtime only. Capture or locate a static spec before modelling its API.
5. `b2c-website-whitelabel-flights` `docs/openapi/bff.yaml` names Account and WAPI as upstreams. Treat that as a document claim until the BFF client code is read. Decide whether `tools/understand-anything` belongs in the model.
6. `ts-tcc` and `ts-web-api` depend on `com.travelstart` libraries (`ts-common` / `ts-commons`, `ts-api-client` / `api-client`, `ota`, `ts-web-common`) that are not in this workspace. Do not model those artifacts as services.
