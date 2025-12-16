# Jardis DbSchema
![Build Status](https://github.com/jardisCore/dbschema/actions/workflows/ci.yml/badge.svg)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![PHP Version](https://img.shields.io/badge/PHP-%3E%3D8.2-777BB4.svg)](https://www.php.net/)
[![PSR-4](https://img.shields.io/badge/autoload-PSR--4-blue.svg)](https://www.php-fig.org/psr/psr-4/)
[![PSR-12](https://img.shields.io/badge/code%20style-PSR--12-blue.svg)](https://www.php-fig.org/psr/psr-12/)
[![Coverage](https://img.shields.io/badge/coverage->95%25-brightgreen)](https://github.com/jardiscore/dbschema)

> **Master your database schema with confidence.** A powerful, developer-friendly PHP library that speaks fluently with MySQL/MariaDB, PostgreSQL, and SQLite – all through one elegant interface.

---

## Why Jardis DbSchema?

Working with multiple databases shouldn't mean writing different code for each one. **Jardis DbSchema** eliminates the complexity by providing a unified, PDO-based solution that adapts to your database automatically.

### 🚀 What Makes It Special

- **🎯 Zero Configuration** – Just pass a PDO connection, and it figures out the rest
- **🔄 Universal Interface** – Write once, run on MySQL, PostgreSQL, or SQLite
- **📊 Complete Schema Insight** – Tables, columns, indexes, foreign keys – everything at your fingertips
- **🛡️ Type-Safe & Validated** – Built with PHP 8.2+, strict types, and PHPStan level 8
- **⚡ SQL DDL Export** – Generate complete database schemas with automatic dependency resolution
- **📦 JSON Export** – Export schema metadata as JSON or PHP arrays
- **🎨 Clean Architecture** – PSR-4, PSR-12, fully tested with 114 tests and 988 assertions
- **🧩 Framework Agnostic** – Use it anywhere: Laravel, Symfony, standalone projects

### 💡 Perfect For

- **Schema migrations** and version control
- **Database documentation** generators
- **ORM and query builder** development
- **Multi-database applications** with consistent behavior
- **Schema comparison** and diff tools
- **DDL export** and database cloning
- **API schema documentation** with JSON export

---

## Overview

This package provides a unified API to consistently retrieve and export schema information from different database systems. With automatic driver detection and normalized output, you can focus on building features instead of wrestling with database quirks.

**Core Features:**

- Unified retrieval of schema information (tables, columns, indexes, foreign keys)
- Standardization of data structures across different database systems
- Field filtering with preserved order
- Type mapping from database types to PHP types
- SQL DDL export with automatic dependency resolution
- JSON/Array export for documentation and API integration
- High testability and interchangeability

---

## Requirements

- **PHP** >= 8.2
- **ext-pdo** (PDO extension)
- **jardispsr/dbschema** ^1.0 (PSR interfaces)
- **jardiscore/dotenv** ^1.0

---

## Supported Database Types

The package includes schema reader implementations for the following database types:

- **MySQL / MariaDB** → `JardisCore\DbSchema\Reader\MySqlReader`
- **PostgreSQL** → `JardisCore\DbSchema\Reader\PostgresReader`
- **SQLite** → `JardisCore\DbSchema\Reader\SqLiteReader`

Each reader class implements `JardisPsr\DbSchema\DbSchemaReaderInterface` and handles database-specific schema queries, returning results in consistent, normalized structures.

---

## Installation

Using Composer:

```bash
composer require jardiscore/dbschema
```

For local development with Docker:

```bash
git clone https://github.com/jardiscore/dbschema.git
cd dbschema
make start      # Start database containers
make install    # Install dependencies
```

---

## Quick Start

### Reading Schema Information

```php
use JardisCore\DbSchema\DbSchemaReader;
use PDO;

// Create PDO connection
$pdo = new PDO('mysql:host=localhost;dbname=mydb', 'user', 'pass');
$reader = new DbSchemaReader($pdo);

// Retrieve all tables
$tables = $reader->tables();
print_r($tables);
// Returns: [['name' => 'users', 'type' => 'BASE TABLE'], ...]

// Retrieve all columns of a table
$columns = $reader->columns('users');
print_r($columns);
// Returns: [
//   ['name' => 'id', 'type' => 'int', 'nullable' => false, 'primary' => true, ...],
//   ['name' => 'name', 'type' => 'varchar', 'length' => 255, ...],
//   ...
// ]

// Retrieve specific columns (order preserved!)
$specificColumns = $reader->columns('users', ['id', 'name', 'email']);
print_r($specificColumns);
// Returns columns in the exact order specified: id, name, email

// Retrieve indexes of a table
$indexes = $reader->indexes('users');
print_r($indexes);

// Retrieve foreign keys of a table
$foreignKeys = $reader->foreignKeys('orders');
print_r($foreignKeys);

// Get PHP type mapping for a database type
$phpType = $reader->fieldType('VARCHAR');
echo $phpType; // "string"

$phpType = $reader->fieldType('int');
echo $phpType; // "int"
```

### Exporting Schema

The unified `DbSchemaExporter` provides multiple export formats with automatic dialect detection:

```php
use JardisCore\DbSchema\DbSchemaReader;
use JardisCore\DbSchema\DbSchemaExporter;
use PDO;

// Create PDO connection and reader
$pdo = new PDO('mysql:host=localhost;dbname=mydb', 'user', 'pass');
$reader = new DbSchemaReader($pdo);

// Create exporter (automatically detects database type from reader)
$exporter = new DbSchemaExporter($reader);

// Export as SQL DDL
$sql = $exporter->toSql(['users', 'orders']);
file_put_contents('schema.sql', $sql);

// Export as JSON (compact)
$json = $exporter->toJson(['users', 'orders']);
echo $json;

// Export as JSON (pretty-printed for documentation)
$jsonPretty = $exporter->toJson(['users', 'orders'], prettyPrint: true);
file_put_contents('schema.json', $jsonPretty);

// Export as PHP array
$array = $exporter->toArray(['users', 'orders']);
print_r($array);
```

---

## Examples

### SQL DDL Export

Export complete database schemas as SQL DDL scripts with automatic table dependency resolution:

```php
use JardisCore\DbSchema\DbSchemaReader;
use JardisCore\DbSchema\DbSchemaExporter;
use PDO;

$pdo = new PDO('mysql:host=localhost;dbname=mydb', 'user', 'pass');
$reader = new DbSchemaReader($pdo);
$exporter = new DbSchemaExporter($reader);

// Export specific tables (automatically sorted by foreign key dependencies)
$ddl = $exporter->toSql(['users', 'orders', 'products']);

// The DDL includes:
// - DROP TABLE statements (reverse dependency order)
// - CREATE TABLE statements (correct dependency order)
// - CREATE INDEX statements
// - ALTER TABLE ADD FOREIGN KEY statements
// - Wrapped in transaction (BEGIN/COMMIT)

file_put_contents('schema.sql', $ddl);
```

The exporter provides:
- **Automatic dependency resolution**: Tables are ordered correctly based on foreign key relationships
- **Complete DDL generation**: DROP TABLE, CREATE TABLE, CREATE INDEX, and ALTER TABLE statements
- **Transactional output**: Wrapped in BEGIN/COMMIT for safe execution
- **Database-specific dialects**: Automatically detects and uses MySqlDialect, PostgresDialect, or SqLiteDialect

### JSON Export

Export schema metadata as JSON for documentation, API schemas, or tooling:

```php
use JardisCore\DbSchema\DbSchemaReader;
use JardisCore\DbSchema\DbSchemaExporter;
use PDO;

$pdo = new PDO('pgsql:host=localhost;dbname=mydb', 'user', 'pass');
$reader = new DbSchemaReader($pdo);
$exporter = new DbSchemaExporter($reader);

// Export as compact JSON
$json = $exporter->toJson(['users', 'orders']);
echo $json;

// Export as pretty-printed JSON
$jsonPretty = $exporter->toJson(['users', 'orders'], prettyPrint: true);
file_put_contents('api-schema.json', $jsonPretty);

// Result structure:
// {
//   "version": "1.0",
//   "generated": "2025-12-16 15:30:00",
//   "tables": {
//     "users": {
//       "columns": [...],
//       "indexes": [...],
//       "foreignKeys": [...]
//     },
//     "orders": {...}
//   }
// }
```

### Array Export

Export schema as PHP array for programmatic processing:

```php
use JardisCore\DbSchema\DbSchemaReader;
use JardisCore\DbSchema\DbSchemaExporter;
use PDO;

$pdo = new PDO('sqlite::memory:');
$reader = new DbSchemaReader($pdo);
$exporter = new DbSchemaExporter($reader);

// Export as array
$schema = $exporter->toArray(['users']);

// Process schema data
foreach ($schema['tables'] as $tableName => $tableData) {
    echo "Table: $tableName\n";

    foreach ($tableData['columns'] as $column) {
        echo "  - {$column['name']}: {$column['type']}\n";
    }

    foreach ($tableData['foreignKeys'] as $fk) {
        echo "  FK: {$fk['constraintCol']} -> {$fk['refContainer']}.{$fk['refColumn']}\n";
    }
}
```

### Column Information Structure

Each column returned by `columns()` includes:

- **name**: Column name
- **type**: Database data type (e.g., 'varchar', 'int', 'datetime')
- **length**: Maximum length for character types
- **precision**: Numeric precision
- **scale**: Numeric scale
- **nullable**: Boolean indicating if NULL is allowed
- **default**: Default value
- **primary**: Boolean indicating if it's a primary key
- **auto_increment**: Boolean indicating auto-increment status
- **enumValues**: Array of enum values (for MySQL ENUM types)

---

## Architecture

### Interfaces (jardispsr/dbschema)

The package uses PSR interfaces from the `jardispsr/dbschema` package:

```php
// Schema reader interface
interface DbSchemaReaderInterface
{
    public function tables(): ?array;
    public function columns(string $table, ?array $fields = null): ?array;
    public function indexes(string $table): ?array;
    public function foreignKeys(string $table): ?array;
    public function fieldType(string $fieldType): ?string;
}

// Schema exporter interface
interface DbSchemaExporterInterface
{
    public function toSql(array $tables): string;
    public function toJson(array $tables, bool $prettyPrint = false): string;
    public function toArray(array $tables): array;
}
```

### Factory Pattern

The `DbSchemaReader` class acts as a factory that automatically detects the PDO driver and instantiates the appropriate reader:

```php
$pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'password');
$reader = new DbSchemaReader($pdo);

// Automatically returns MySqlReader, PostgresReader, or SqLiteReader internally
$tables = $reader->tables();
```

### Reader Pattern

The package uses the reader pattern with database-specific implementations that handle:

- Database-specific SQL queries
- Result normalization (boolean values, numeric types)
- Consistent data structures across all database types

### Exporter Architecture

The `DbSchemaExporter` provides a unified interface for multiple export formats:

- **SQL DDL Export**: Uses DdlDialectInterface implementations (MySqlDialect, PostgresDialect, SqLiteDialect)
- **JSON Export**: Uses JsonExporter for structured JSON output
- **Array Export**: Direct access to normalized schema data structures
- **Automatic Dialect Detection**: Detects database type from PDO driver
- **Caching**: Internally caches exporter instances for performance

Internal components:
- **DdlDialectInterface**: Defines database-specific DDL syntax
- **SqlDdlExporter**: Orchestrates DDL generation
- **DependencyResolver**: Performs topological sorting for foreign key dependencies
- **JsonExporter**: Handles JSON serialization with optional pretty-printing

---

## Development

### Docker Setup

The project uses Docker for local development with the following services:

- **MySQL** (mysql_test)
- **MariaDB** (mariadb_test)
- **PostgreSQL** (postgres_test)
- **PHP CLI** (phpcli) - PHP 8.3 with Xdebug

### Available Make Commands

**Docker Management:**
```bash
make start           # Start all database containers
make stop            # Stop and remove all containers
make restart         # Restart all containers
make status          # Show container status
make clean           # Stop containers and clean up volumes
make remove          # Remove all Docker resources
```

**Composer:**
```bash
make install         # Run composer install
make update          # Run composer update
make autoload        # Dump autoload
```

**Quality Assurance:**
```bash
make phpunit                  # Run all tests
make phpunit-coverage         # Run tests with text coverage
make phpunit-coverage-html    # Run tests with HTML coverage
make phpunit-reports          # Run tests with XML reports
make phpstan                  # Run PHPStan (level 8)
make phpcs                    # Run code style checks
```

**Development:**
```bash
make shell           # Open shell in PHP container
make logs            # Show container logs
```

---

## Testing

The project includes comprehensive PHPUnit tests for all database types:

- **114 tests** with **988 assertions**
- **Integration tests**: Testing against real databases (MySQL, MariaDB, PostgreSQL, SQLite)
- **Component tests**: Testing readers, exporters, and dialects
- **Coverage**: >95% code coverage

Run tests:
```bash
make start           # Start databases first
make phpunit         # Run all tests
```

---

## Code Quality

The repository maintains high code quality standards:

* **PHPStan** - Static analysis at level 8
* **PHPCS** - PSR-12 coding standards
* **PHPUnit** - Comprehensive test coverage
* **Pre-commit hooks** - Automated quality checks

### CI/CD Pipeline

GitHub Actions workflow includes:

1. Dependency installation
2. Code style checks (`make phpcs`)
3. Static analysis (`make phpstan`)
4. Test execution with coverage (`make phpunit-coverage`)

---

## License

This project is licensed under the **MIT License** – see `LICENSE` in the repository.

---

## Support

- **Issues**: https://github.com/jardiscore/dbschema/issues
- **Email**: jardisCore@headgent.dev

---

## Credits

**Jardis DbSchema** is developed and maintained by the **Jardis Development Core** team at **Headgent**.

Built with ❤️ for the PHP community.

---

**Ready to simplify your database schema handling?** Install Jardis DbSchema today and experience the difference.

```bash
composer require jardiscore/dbschema
```
