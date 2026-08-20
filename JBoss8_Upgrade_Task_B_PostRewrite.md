# Task B: Post-Rewrite Migration Completion

Run this task only after **Task A** preflight + `rewrite:run` are complete.

---

## 1. Maven / pom.xml Changes

In the main `pom.xml`:

- Replace `<maven.compiler.source>1.8</maven.compiler.source>` and `<maven.compiler.target>1.8</maven.compiler.target>` with:
  ```xml
  <maven.compiler.release>21</maven.compiler.release>
  ```
- If the project already includes a Maven dependency with `<groupId>`/`<artifactId>` of `MISWebAppFramework` and a version of `5` or higher, update that dependency version to `7.02`.

### IntelliJ / Project SDK

- Set the project **SDK** to **Java 21** in IntelliJ Project Structure.
- Keep **Language level** at **SDK default** unless the project needs a specific Java 21 language level override.

### For Spring Framework apps

- Update `<parent>` to:
  ```xml
  <parent>
      <groupId>nc.dit.dps</groupId>
      <artifactId>DpsCoreProject</artifactId>
      <version>5.0.0</version>
  </parent>
  ```
- Bump the project's own `<version>` (not inside `<parent>`) — e.g. `1.0.0` → `2.0.0`.
- Update all Core sub-module version properties to `5.0.0`:
  - `DpsCoreBean.version`, `DpsCoreDao.version`, `DpsCoreWeb.version`, `DpsCoreService.version`, `DpsCoreReports.version`, `DpsCoreUtils.version`, `DpsCoreCache.version`
- Ensure `<remote-mvn-repository>` and `<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>` are present.
- Remove or comment out now-unneeded root pom properties: `cdi.version`, `reactivestreams.version`, `infinispan*` versions, `jgroups.version`, `fasterxml.version`.
- In each **module** pom: update `<parent>` version and any standalone `<version>` to the new project version.
- Remove duplicate properties from module poms that are inherited from the parent; delete the `<properties>` section entirely if it becomes empty.
- Trim each module's dependency list to only what it actually needs.
- Use property references instead of hardcoded versions (e.g. `${DpsCoreBean.version}`).
- For intra-project module dependencies, reference version as `${project.version}`:
  ```xml
  <dependency>
      <groupId>nc.dit.dps.rnaplus</groupId>
      <artifactId>dao</artifactId>
      <version>${project.version}</version>
  </dependency>
  ```

### Shared framework/jar version targets
| Jar | Target Version |
|---|---|
| OpusSecurity | 6.0.0 (JBoss8 branch) |
| DOCProperties | 3.0.0 |
| GZipCompressionFilter | 2.0 |
| DOCEmailService | 2.0 |
| DOCRestFramework | 4.0.0 (JBoss8 branch) |
| iBATISFramework | 4.0.0 (JBoss8 branch) |
| JDBCLoggingProxy | 10.0 (JDK21 branch) |
| DPS Core | 5.0.0 (SB_3-0-13_jdk21 branch) |

---

## 2. Config File Changes

### jboss-web.xml — security domain
In all `jboss-web.xml` files, update:
```xml
<security-domain flushOnSessionInvalidation="true">java:/jaas/OpusServices</security-domain>
```
to:
```xml
<security-domain flushOnSessionInvalidation="true">opus-domain</security-domain>
```

### jboss-deployment-structure.xml (Spring apps)
Add or replace this file for each Spring app:
```xml
<jboss-deployment-structure xmlns="urn:jboss:deployment-structure:1.2">
    <deployment>
        <dependencies>
            <module name="db2jdbc_library.DB2Drivers"/>
            <module name="org.apache.commons.collections" export="true"/>
            <module name="jakarta.websocket.api" export="true"/>
            <module name="java.management" export="true"/>
            <module name="us.nc.doc.opus.security" export="true"/>
        </dependencies>
        <exclude-subsystems>
            <subsystem name="logging"/>
        </exclude-subsystems>
        <exclusions>
            <module name="org.infinispan" slot="main"/>
            <module name="org.infinispan.commons" slot="main"/>
            <module name="org.springframework.spring" slot="main"/>
        </exclusions>
    </deployment>
</jboss-deployment-structure>
```
- [ ] Add/update this file for each Spring app.

### web.xml — namespace/schema update
Update the `<web-app>` opening tag to Jakarta EE 6.0:
```xml
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="https://jakarta.ee/xml/ns/jakartaee"
         xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee https://jakarta.ee/xml/ns/jakartaee/web-app_6_0.xsd"
         version="6.0">
```

### application.properties additions (Spring web module)
Add:
```
infinispan.embedded.metrics.enabled=false
management.health.db.enabled=false
spring.autoconfigure.exclude=org.springframework.boot.actuate.autoconfigure.metrics.jersey.JerseyServerMetricsAutoConfiguration
spring.autoconfigure.exclude=org.infinispan.spring.starter.embedded.actuator.InfinispanCacheMetricsAutoConfiguration,\
 org.springframework.boot.actuate.autoconfigure.metrics.jersey.JerseyServerMetricsAutoConfiguration
infinispan.metrics.enabled=false
spring.infinispan.embedded.metrics.enabled=false
management.metrics.enable.infinispan=false
management.endpoint.prometheus.enabled=false
management.prometheus.metrics.export.enabled=false
```

Also add Micrometer dependency to web module `pom.xml` if missing:
```xml
<dependency>
   <groupId>io.micrometer</groupId>
   <artifactId>micrometer-core</artifactId>
</dependency>
```

---

## 3. Code Changes (Spring Boot 3 Specific)

### Current logged-in staff ID
Replace direct `OpusPrincipal` / `getDetails()` usage with `CurrentStaffIDHelper`.

### Web Security customizer
Replace `.antMatchers(...)` with `.requestMatchers(...)` in `webSecurityCustomizer`.

### Apache HttpClient upgrade (4.x → 5.x)
Replace:
```xml
<groupId>org.apache.httpcomponents</groupId>
<artifactId>httpclient</artifactId>
<version>4.5.14</version>
```
with:
```xml
<groupId>org.apache.httpcomponents.client5</groupId>
<artifactId>httpclient5</artifactId>
<version>5.2.3</version>
```
Update imports to `org.apache.hc.client5.*` and API usage as needed.

### Freemarker (.ftl) security tags
In `Header.ftl`, remove:
```
<#assign security=JspTaglibs["http://www.springframework.org/security/tags"]/>
```
Then map all Spring Security JSP taglib usage to the new `security` object pattern and replace `</@security.authorize>` with `</#if>`.

---

## 4. Final Verification

Confirm by search:

- No remaining `javax.*` references in Java source or `WEB-INF/jboss*.xml` files.
- `jboss-web.xml` security domain is `opus-domain` (not `java:/jaas/OpusServices`).
- `web.xml` uses the Jakarta EE 6.0 namespace.
- No remaining `.antMatchers(` calls in Spring Security configuration.
- No remaining `org.apache.httpcomponents:httpclient` 4.x imports.
- No remaining Spring Security JSP taglib assignments in `.ftl` files.
- `maven.compiler.release` is `21`; `maven.compiler.source/target` are removed.

Report a summary of changes and any manual follow-up still required.
