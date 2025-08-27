# Dinamail — Plan de réalisation

## 0) Vue d’ensemble

* **Pile** : Java 21, Spring Boot 3.x, Spring Security (OAuth2 Resource Server), Maven, MySQL 8, JPA/Hibernate, JHipster (UI React), MapStruct, Lombok, Flyway, Testcontainers, RestAssured, Micrometer/Prometheus, OpenAPI v3, (Optionnel V1.1) ShedLock pour verrous de jobs.
* **Repos** : mono-repo Git avec deux modules Maven : `mail-sender/` et `recup-mail/` + `infra/` (compose), `docs/` (OpenAPI/JDL).
* **Environnements** : `dev`, `staging`, `prod` (profils Spring + fichiers de conf séparés).

## 1) Structure du dépôt

```
/ (root)
 ├─ mail-sender/
 │   ├─ pom.xml
 │   ├─ src/main/java/... (domain, repository, service, web)
 │   ├─ src/main/resources/
 │   │   ├─ application.yml
 │   │   ├─ application-dev.yml
 │   │   ├─ application-staging.yml
 │   │   └─ db/migration (Flyway)
 │   └─ src/test/java/...
 ├─ recup-mail/
 │   ├─ pom.xml
 │   ├─ src/main/java/...
 │   ├─ src/main/resources/
 │   │   ├─ application.yml
 │   │   ├─ application-dev.yml
 │   │   └─ db/migration (Flyway)
 │   └─ src/test/java/...
 ├─ docs/
 │   ├─ openapi/
 │   │   ├─ openapi-mail-sender.yaml
 │   │   └─ openapi-recup-mail.yaml
 │   ├─ model/dinamail.jdl
 │   └─ diagrams/*.mmd
 ├─ infra/
 │   ├─ docker-compose.yml (MySQL, Prometheus, Grafana, MailHog)
 │   └─ grafana-dashboards/
 ├─ .github/workflows/
 │   └─ ci.yml
 └─ pom.xml (parent)
```

## 2) Initialisation & bootstrap

### 2.1 Parent Maven

```xml
<!-- pom.xml (root) -->
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.dinamail</groupId>
  <artifactId>dinamail</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <packaging>pom</packaging>
  <modules>
    <module>mail-sender</module>
    <module>recup-mail</module>
  </modules>
  <properties>
    <java.version>21</java.version>
    <spring.boot.version>3.3.0</spring.boot.version>
  </properties>
</project>
```

### 2.2 Génération mail-sender via JHipster

* Installer JHipster (local) puis **import JDL** `docs/model/dinamail.jdl`.
* Configurer **authenticationType: oauth2**, **databaseType: sql**, **prod: mysql**.
* Déplacer le code dans le module `mail-sender/` si généré ailleurs.

### 2.3 Création recup-mail (Spring Initializr)

* Dépendances : Web, OAuth2 Resource Server, JPA, MySQL, Validation, Actuator, OpenAPI, Flyway, Lombok, MapStruct.

## 3) Tâches détaillées — mail-sender (V1)

### 3.1 Modèle & persistence

*

### 3.2 REST API

*

### 3.3 Service d’envoi

*

### 3.4 Sécurité

*

### 3.5 UI JHipster

*

### 3.6 Observabilité

*

### 3.7 Tests

*

## 4) Tâches détaillées — recup-mail (V1)

### 4.1 Modèle & persistence

*

### 4.2 Connecteurs & collecte

*

### 4.3 Sécurité & UI minimale

*

### 4.4 Observabilité & tests

*

## 5) Sécurité & conformité

* **OAuth2/JWT** : issuer configurable (Keycloak/CAS gateway plus tard). Scopes: `email:read`, `email:write`, `job:admin`.
* **Secrets** : placeholder `secretAlias` en V1 ; intégration Vault en V2.
* **RGPD** : TTL de contenu (propriété `dinamail.retention.days`), job purge planifié (supprimer `corps`/PJ, conserver méta).

## 6) CI/CD (GitHub Actions — `ci.yml`)

* **Jobs** : `build-test` (Java 21), `docker-images`, `deploy-staging` (optionnel), `scan` (OWASP/Trivy).
* **Étapes** :

  1. Checkout
  2. Setup Java 21
  3. Cache Maven
  4. Lint (Spotless/Checkstyle)
  5. Tests (unitaires + intégration avec Testcontainers)
  6. Build JARs
  7. Build & push images Docker (`mail-sender`, `recup-mail`)

Extrait :

```yaml
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { distribution: 'temurin', java-version: '21' }
      - name: Cache Maven
        uses: actions/cache@v4
        with:
          path: ~/.m2/repository
          key: maven-${{ hashFiles('**/pom.xml') }}
      - name: Build & Test
        run: mvn -B -ntp verify
```

## 7) Déploiement local & dev

### 7.1 Docker Compose (`infra/docker-compose.yml`)

* Services : `mysql`, `mailhog` (smtp + ui), `prometheus`, `grafana`.
* Monter `mail-sender`/`recup-mail` en local (profiles `dev`) avec SMTP pointant sur MailHog.

### 7.2 Config application (extraits)

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/dinamail?useSSL=false&allowPublicKeyRetrieval=true
    username: dinamail
    password: dinamail
  jpa:
    hibernate:
      ddl-auto: validate

spring.security.oauth2.resourceserver.jwt.issuer-uri: http://localhost:8081/realms/dinamail

mail:
  smtp:
    host: localhost
    port: 1025 # MailHog
```

## 8) Qualité & conventions

* **Style** : Checkstyle/Spotless (Google Java Style), SonarLint local.
* **DTO/Mapper** : MapStruct entre entités et API models.
* **Logs** : JSON (logback encoder) + champs `traceId`.
* **Erreurs API** : RFC7807 (application/problem+json) via `@ControllerAdvice`.

## 9) Découpage en tickets (V1)

### Épic 1 — Bootstrap & infra

*

### Épic 2 — mail-sender

*

### Épic 3 — recup-mail

*

### Épic 4 — Pilote & finitions

*

## 10) Critères de done (DoD)

* **API** : Contrats OpenAPI exposés sur `/v3/api-docs` + Swagger UI.
* **Tests** : couverture > 80% sur services, tests d’intégration REST OK (Testcontainers).
* **Idempotence** : doublon `(sourceApp,idFonctionnel)` testé (409 ou renvoi même id).
* **Envoi SMTP** : démonstration via MailHog + message visible UI.
* **Observabilité** : métriques visibles dans Prometheus, dashboard Grafana importé.

## 11) Roadmap V1 → V2

* **V1.1** : EmailEvent UI timeline, relance/annulation, export CSV.
* **V2** : Fallback provider API, webhook bounces → suppression list, templates (Mustache/Thymeleaf) + i18n, pièces jointes, Vault, Kafka/RabbitMQ, CAS SSO.

## 12) Snippets clés

### 12.1 Scheduler d’envoi

```java
@Scheduled(fixedDelayString = "${dinamail.sender.poll-ms:5000}")
public void processPending() {
  List<Email> batch = emailRepository.findTop50ByStatusOrderByCreatedAtAsc(PENDING);
  for (Email e : batch) {
    try {
      e.setStatus(SENDING);
      emailRepository.save(e);
      var result = mailGateway.send(mapper.toDto(e));
      if (result.success()) {
        e.setStatus(SENT);
      } else {
        e.setStatus(FAILED);
        e.setAttemptCount(e.getAttemptCount()+1);
        e.setErrorMessage(result.providerMessage());
      }
      e.setLastAttemptAt(Instant.now());
      emailRepository.save(e);
      emailEventService.log(e, result.success()?"SENT":"FAILED", result.providerMessage());
    } catch (Exception ex) {
      // idem FAILED
    }
  }
}
```

### 12.2 Security Resource Server (`application.yml`)

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: ${OAUTH_ISSUER_URI}
```

### 12.3 Idempotency Filter (pseudo-code)

```java
if (request.hasHeader("Idempotency-Key")) {
  if (idemStore.contains(key)) return 409; // ou renvoyer la ressource existante
  idemStore.put(key, now);
}
```

## 13) Démarrage rapide (Makefile)

```make
.PHONY: dev
init:
mvn -q -ntp -DskipTests package

up:
docker compose -f infra/docker-compose.yml up -d

down:
docker compose -f infra/docker-compose.yml down -v

dev:
mvn -pl mail-sender -am spring-boot:run & \
mvn -pl recup-mail -am spring-boot:run
```

---

**Prêt pour exécution** : Codex peut créer le parent Maven, initialiser les deux modules, importer le JDL, et commencer par l’épic 2 (mail-sender) en parallèle de l’épic 1 (infra/CI).
