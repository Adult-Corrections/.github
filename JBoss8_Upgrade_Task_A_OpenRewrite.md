# Task A: OpenRewrite Gate + Automated Jakarta Migration

You are migrating this Java web application from JBoss 7 (JDK 8) to JBoss 8 (JDK 21 / Jakarta EE).
This task is only for preflight checks and running OpenRewrite.

## 0. Java 21 Toolchain Setup + Preflight Gate (must pass before any migration edits)

Before running checks, switch the current shell to Java 21.

### 0.1 Set Java 21 in your shell

**Check if Java 21 is installed:**

Verify that this folder exists on your system:
```
C:\openjdk-21.0.9.0.10-1
```

If this folder does not exist, Java 21 is not installed and must be installed before proceeding.

**Set Java 21 in your current shell:**

On Windows PowerShell:

```powershell
$env:JAVA_HOME = "C:\openjdk-21.0.9.0.10-1"
$env:Path = "$env:JAVA_HOME\bin;$env:Path"
```

### 0.2 Run preflight checks

Run these checks and stop immediately if any fail:

1. Runtime Java version:
   - `java -version`
   - **Required:** Java 21
2. Maven runtime Java:
   - `mvn -version`
   - **Required:** Maven reports Java 21 (same toolchain expected for project commands)
3. Repository cleanliness check:
   - `git status --short`
   - **Required:** No unexpected dirty files, or user explicitly approves continuing with local changes present
4. OpenRewrite plugin/artifact resolution check (no code change run yet):
   - `mvn -q -DskipTests rewrite:discover`
   - **Required:** Command succeeds and recipe list resolves

If any gate fails, report blockers and do not continue with migration changes.

---

## 1. Identify javax → jakarta Exposure (baseline)

Search for all remaining `javax.*` references in:
- Java source files
- `WEB-INF/jboss*.xml` files

Report findings before running rewrite.

---

## 2. Add OpenRewrite plugin to main pom.xml

Add the following plugin to the **main project `pom.xml`** inside `<build><plugins>` (or merge into existing block):

```xml
<plugin>
   <groupId>org.openrewrite.maven</groupId>
   <artifactId>rewrite-maven-plugin</artifactId>
   <version>6.23.0</version>
   <configuration>
      <activeRecipes>
         <recipe>org.openrewrite.java.migrate.jakarta.JavaxMigrationToJakarta</recipe>
      </activeRecipes>
   </configuration>
   <executions>
      <execution>
         <id>rewrite</id>
         <goals>
            <goal>run</goal>
         </goals>
      </execution>
   </executions>
   <dependencies>
      <dependency>
         <groupId>org.openrewrite.recipe</groupId>
         <artifactId>rewrite-migrate-java</artifactId>
         <version>3.0.1</version>
      </dependency>
   </dependencies>
</plugin>
```

---

## 3. Run rewrite and review

1. Run:
   - `mvn rewrite:run`
2. Review local changes carefully:
   - IDE Local Changes or `git diff`
3. Search `WEB-INF/jboss*.xml` for remaining `javax.*` references and fix any misses manually.

---

## 4. Disable rewrite plugin after successful run

Comment out or remove the OpenRewrite plugin block so it does not run again automatically.

---

## Exit criteria for Task A

- Preflight gate passed
- `rewrite:run` completed successfully
- Rewrite changes reviewed
- `WEB-INF/jboss*.xml` `javax.*` misses addressed
- Rewrite plugin commented out or removed
