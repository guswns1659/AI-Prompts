## Initializing a Spring Boot Project with Gradle

This document guides you on how to set up a Spring Boot project with a single-project structure using `gradle init`.

### 1. Create and Restructure Gradle Project

1.  Create a new Java application using the `gradle init` command.

    ```bash
    gradle init --type java-application --dsl kotlin --project-name <project-name> --package <package-name> --overwrite
    ```

2.  Change the multi-project structure created by `gradle init` to a single-project structure. First, move the contents of the `app` directory (`src`, `build.gradle.kts`) to the root directory.

    ```bash
    mv app/src .
    mv app/build.gradle.kts .
    ```

3.  Delete the empty `app` directory.

    ```bash
    rmdir app
    ```

4.  Remove the `include("app")` line from the `settings.gradle.kts` file.

### 2. Spring Boot Configuration

1.  **Remove Default `application` Plugin**

    The `gradle init` command adds the `application` plugin by default. Since we will use Spring Boot, this needs to be removed.

    *   In `build.gradle.kts`, delete the `application` line from the `plugins {}` block.
    *   Delete the entire `application {}` configuration block.

2.  **Configure Version Catalog (`gradle/libs.versions.toml`)**

    Add the Spring Boot plugin and desired starters to the version catalog. This allows for centralized dependency management.

    ```toml
    [versions]
    springBoot = "3.3.0"
    springDependencyManagement = "1.1.5"

    [libraries]
    # Spring Boot Starters
    spring-boot-starter-web = { module = "org.springframework.boot:spring-boot-starter-web" }
    spring-boot-starter-test = { module = "org.springframework.boot:spring-boot-starter-test" }

    [plugins]
    spring-boot = { id = "org.springframework.boot", version.ref = "springBoot" }
    spring-dependency-management = { id = "io.spring.dependency-management", version.ref = "springDependencyManagement" }
    ```

3.  **Configure Build Script (`build.gradle.kts`)**

    Apply the necessary plugins and add the dependencies from the version catalog.

    ```kotlin
    import org.gradle.jvm.toolchain.JavaLanguageVersion

    plugins {
        java // Required for Java compilation
        alias(libs.plugins.spring.boot)
        alias(libs.plugins.spring.dependency.management)
    }

    repositories {
        mavenCentral()
    }

    dependencies {
        implementation(libs.spring.boot.starter.web)
        testImplementation(libs.spring.boot.starter.test)
        testRuntimeOnly("org.junit.platform:junit-platform-launcher")
    }

    java {
        toolchain {
            languageVersion.set(JavaLanguageVersion.of(21))
        }
    }

    tasks.named<Test>("test") {
        useJUnitPlatform()
    }
    ```

4.  **Create Main Application Class (`App.java`)**

    Modify the main application class to be a Spring Boot application.

    ```java
    package com.example.jackpangecommerce;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;

    @SpringBootApplication
    public class App {
        public static void main(String[] args) {
            SpringApplication.run(App.class, args);
        }
    }
    ```

5.  **Clean Up Test Files**

    The default `AppTest.java` created by `gradle init` will no longer be valid. Delete it or modify it for Spring Boot testing.

    ```bash
    rm src/test/java/com/example/jackpangecommerce/AppTest.java
    ```

# GEMINI.md — Project Execution Rules

## Core Directive

All actions, reviews, and code generation must strictly follow the specifications and requirements defined in **prd.md** and **tasklist.md**.
If any instruction or request conflicts with prd.md or tasklist.md, always prioritize prd.md and tasklist.md.

## Behavior Guidelines

* **Clarity**: Always respond with clear, step-by-step instructions or code that can be copied and executed directly.
* **Consistency**: Use **Java + Spring Boot + Gradle** for all backend implementations. Do not switch to other languages or frameworks.
* **Scope Control**: Do not add features or complexity beyond prd.md and tasklist.md unless explicitly requested.
* **Result First**: Prioritize producing a working result quickly. Defer optimizations, refactoring, or enhancements unless they are critical for correctness.
* **Task Mapping**: For every change or recommendation, map it explicitly to the relevant section of prd.md or tasklist.md so developers know its origin.

## Build & Dependency Management

*   **Gradle Version Catalog**: Version management in Gradle is handled using a Version Catalog via the `gradle/libs.versions.toml` file.
*   **Automatic Recognition**: `libs.versions.toml` is automatically recognized by Gradle, so it should not be redundantly declared in `settings.gradle.kts` using a `versionCatalogs` block.
*   **Linting & Formatting**: Code formatting is managed by the "com.diffplug.spotless" plugin, and static analysis is done by the "pmd" plugin.
*   **Trailing Commas**: Trailing commas are enforced using `.editorconfig` with the `ij_kotlin_allow_trailing_comma` and `ij_kotlin_allow_trailing_comma_on_call_site` properties.

## Additional Review & Action Rules

1. **Code Quality**

   * Ensure code compiles and runs locally without external dependencies beyond those listed in tasklist.md.
   * Apply modern Java style and best practices, ensuring code adheres to the configured Spotless (Google Java Format) and PMD rules.
   * If PMD reports a Cognitive Complexity violation, actively refactor the code to reduce its complexity.

2. **Documentation**

   * Each deliverable must include a short inline comment explaining purpose, aligned with prd.md requirements.
   * Update README if a new command, dependency, or setup step is introduced.

3. **Testing Discipline**

   * For every new API, provide at least one unit test.
   * Use clear, deterministic test data consistent with seed data described in prd.md.
   * If a unit test fails more than twice, do not create that unit test to avoid getting stuck.
   * Use JUnit 5 as the testing framework.
   * Use AssertJ for assertions.
   * Use Mockito for mocking in Java code.

4. **Error Handling**

   * Validate inputs according to prd.md (e.g., minimum query length).
   * Return structured error responses, never raw stack traces.

5. **Security & Safety**

   * Do not hard-code secrets or credentials.
   * Keep endpoints safe from abuse: enforce validation and mention rate-limiting strategy as a TODO if not implemented.

6. **Feedback Style**

   * Use concise, actionable feedback: describe the issue, why it matters, and how to fix it.
   * Highlight good practices found in code (e.g., correct use of Kotlin idioms, clean separation of service and controller).

---
# GEMINI.md — Commit Planning and Message Rules

## Core Directive
• All commit planning and messages must be based strictly on prd.md and tasklist.md.
• Do not squash. Create multiple small, atomic commits grouped by dependency and execution order.
• Each commit must compile and pass tests on its own (buildable, revert-friendly).

## Grouping & Order (top-down dependency)

1. Build/Config first: Gradle build files, dependencies, application.yml, docker-compose, CI scripts.
2. Database/Index next: Elasticsearch index/mapping creation, seeders, migrations.
3. Domain/Model: data classes, DTOs, @Document entities.
4. Repository layer: ElasticsearchRepository, query adapters.
5. Service layer: business logic (SuggestService).
6. Web layer: Controllers, exception handlers, request validation.
7. Frontend assets: static index.html, basic JS debounce.
8. Tests: integration tests (API/ES).
9. Docs: README, runbook updates.

## Splitting Principles
• One reason to change per commit (Single Responsibility).
• Separate refactor vs behavior change.
• Do not mix config changes with business logic.
• Large files: split by logical sections if they affect different layers.
• If a change depends on another, the dependency must appear in an earlier commit.

## Commit Message Style (Conventional Commits)
• Format: type(scope): summary ≤ 72 chars
• Types: feat, fix, refactor, chore, docs, test, build, ci
• Body: what changed, why it matters, context (link to prd.md/tasklist.md section).
• Footer: BREAKING CHANGE (if any), issue refs.
• 모든 커밋 메시지는 한글로 작성합니다.

## Required Content in Each Message Body
• Motivation: reference the exact prd.md/tasklist.md item (e.g., “PRD §2.1 Autocomplete API”).
• Implementation note: key design choice or trade-off.
• Validation: how to run/verify locally (command or curl).
• Risk: any follow-ups or TODOs explicitly listed.

## Output Format (what you must produce before committing)

1. Commit Plan (ordered):

   * A numbered list of commits.
   * For each commit: title, rationale, impacted files (paths), dependencies (previous commit numbers).
2. Commit Messages:

   * For each planned commit: final Conventional Commit header + body + footer.
3. Git Command Block:

   * For each commit: exact git add paths and git commit -m commands.
   * Ask for user confirmation before executing the commands.

## Quality Gates per Commit
• Must compile: ./gradlew build (or at least \:app\:compileKotlin).
• Tests: run relevant tests (unit for service, integration for API) if present.
• Lint/style: keep Kotlin idioms (nullable safety, data classes).
• No stack traces or secrets in commit messages.

## Examples (for this project)

1. build(config): initialize Spring Boot + Elasticsearch deps
   Body: Add spring-boot-starter-web, validation, actuator, spring-data-elasticsearch.
   Verify: ./gradlew build

2. build(es): docker-compose for Elasticsearch (single-node)
   Body: Add compose file; xpack.security.disabled=true for local.
   Verify: docker compose up -d && curl :9200

3. feat(index): create items index with search\_as\_you\_type
   Body: IndexInitializer creates items with title as search\_as\_you\_type.
   Verify: curl GET :9200/items

4. feat(seed): bulk seed sample KO/EN items
   Body: CommandLineRunner with 50–200 docs.
   Verify: curl GET :9200/items/\_count

5. feat(domain): add Item entity and repository
   Body: @Document(indexName = "items"), ItemRepository.
   Verify: repository saves/reads

6. feat(service): SuggestService with bool\_prefix query
   Body: q length ≥ 2, limit default 8, optional lang filter, popularity sort.
   Verify: ./gradlew test (service unit tests)

7. feat(api): GET /api/search/suggest controller + validation
   Body: returns {query, suggestions, tookMs}, 400 on short q.
   Verify: curl "[http://localhost:8080/api/search/suggest?q=co](http://localhost:8080/api/search/suggest?q=co)"

8. feat(ui): static page with 200 ms debounce
   Body: index.html shows suggestions; keyboard ↑↓, Enter.
   Verify: open [http://localhost:8080](http://localhost:8080)

9. test(api): integration tests for suggest endpoint
   Body: MockMvc tests for 200/400, size/ordering.
   Verify: ./gradlew test

10. docs(readme): local run instructions
    Body: docker compose up; ./gradlew bootRun; open /
    Verify: check README steps

## Commit Message Templates
### Header
type(scope): concise summary

### Body

* Context: reference to prd.md/tasklist.md section.
* Why: rationale and dependency note.
* How to verify: exact command (build/test/curl).
* Notes: risks/TODOs.

### Footer

* BREAKING CHANGE: … (if any)
* References: issue/ticket

## Review Checklist for Commit Plan
\[ ] Dependency order respected (config → data → domain → repo → service → web → ui → tests → docs)
\[ ] Each commit builds independently
\[ ] Tests introduced in same or later commit never rely on uncommitted code
\[ ] Messages are Conventional, ≤ 72-char headers, meaningful bodies
\[ ] No unrelated changes mixed in a single commit

## Failure Handling
• If a commit fails build/tests, split the change or move dependent parts into earlier commits.
• If a file includes both refactor and feature changes, split the file changes across two commits.

This is the end of the commit planning and messaging rules.aging rules.

---
## Code Quality Check Workflow

When the user asks to "check code quality" (or "코드 퀄리티 체크해줘"), follow these steps. This workflow uses the `spotless` plugin for formatting and `pmd` for static analysis.

1.  **Apply Code Formatting**: Execute `./gradlew spotlessApply` to automatically format all Java source code according to the configured style (e.g., Google Java Format).
2.  **Check Code Formatting**: Execute `./gradlew spotlessCheck` to verify that all files are correctly formatted. This is useful for CI environments to fail the build if the code is not formatted.
3.  **Run Static Analysis**: Execute `./gradlew pmdMain` to run PMD analysis, which checks for potential bugs, dead code, and suboptimal code, including complexity analysis.
4.  **Analyze Reports**: Check the generated reports in `build/reports/` (e.g., `build/reports/spotless/`, `build/reports/pmd/main.html`) to review any identified issues.
5.  **Propose Refactoring**: Based on the reports, propose specific refactoring solutions to improve code quality, reduce complexity, and fix potential bugs.
