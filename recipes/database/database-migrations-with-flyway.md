# Database Migrations with Flyway

- [Database Migrations with Flyway](#database-migrations-with-flyway)
  - [Introduction](#introduction)
  - [Adding Flyway to Your Project](#adding-flyway-to-your-project)
  - [Usage](#usage)
    - [Running Migrations](#running-migrations)
  - [Configuring Flyway](#configuring-flyway)

## Introduction

Database migrations help manage changes to a database schema over time. They allow you to version control your schema, ensuring changes are applied in a consistent and repeatable way. [Flyway](https://flywaydb.org/) is a popular migration tool that integrates seamlessly with Spring Boot and simplifies the process of maintaining a database schema alongside your application code.

Think of Flyway as **version control for your database schema**. By automating the migration process, Flyway helps keep your schema in sync with your application code, making it easier for team members to stay updated and avoid conflicts.

## Adding Flyway to Your Project

To use Flyway, add the Flyway dependency to your project. Place the following snippet in your `pom.xml` file:

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

Additionally, ensure that your project has the appropriate database driver. For instance, if you’re using MySQL or PostgreSQL, add one of the following dependencies to your `pom.xml`:

**For MySQL:**
```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-mysql</artifactId>
</dependency>
```

**For PostgreSQL:**
```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-database-postgresql</artifactId>
</dependency>
```

Once these dependencies are added, Flyway is ready to use in your project.

## Usage

By default, Flyway looks for migration scripts in the `db/migration` directory within your project’s classpath. Migration scripts should follow the naming convention `V{version}__{description}.sql`, where:
- `V{version}` represents the version number of the migration.
- `{description}` provides a brief description of the migration.

**Example Migration File Names**:
- `V1__create_users_table.sql`
- `V2__add_email_to_users_table.sql`
- `V3__create_posts_table.sql`
- `V4__add_deleted_at_to_posts_table.sql`
- `V5__add_index_to_posts_table.sql`

Each file can contain SQL statements to create, update, or delete tables or indexes. For instance, `V1__create_users_table.sql` might contain the SQL code to create a `users` table.

### Running Migrations

In Spring Boot, migrations run automatically when the application starts. However, you can also apply migrations manually via the command line:

```bash
mvn flyway:migrate
```

**Additional Flyway Commands**:
- To undo the last migration (available in Flyway Teams):  
  ```bash
  mvn flyway:undo
  ```

- To clean the database, removing all objects created by migrations:
  ```bash
  mvn flyway:clean
  ```

For more commands and details, visit the [Flyway CLI documentation](https://documentation.red-gate.com/fd/commands-184127446.html/).

## Configuring Flyway

Flyway configuration in Spring Boot is done in the `application.yml` file. Here is an example configuration:

```yaml
spring:
  flyway:
    enabled: true
    baseline-on-migrate: true
    locations: classpath:db/migration
    validate-on-migrate: true
```

**Explanation of Properties**:
- `enabled`: When set to `true`, Flyway is enabled. You can set this to `false` if you need to disable Flyway for certain environments, such as testing or production.
  
- `baseline-on-migrate`: This property creates a baseline version of the database if it doesn’t already exist. It’s useful when introducing Flyway into an existing project that has no previous migrations.

- `locations`: Specifies where Flyway looks for migration files. By default, it searches `classpath:db/migration`, but you can customize this as needed.

- `validate-on-migrate`: Validates the checksum of each migration file before applying it. If the checksum of a migration file differs from the checksum of the same file in previous runs, Flyway will throw an error. This helps ensure that migrations haven’t been modified after they were applied.
