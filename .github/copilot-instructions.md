# Wallabag Copilot Instructions

## Repository Overview

**Wallabag** is a self-hosted, web-based read-it-later application written in PHP using the Symfony framework. It allows users to save web pages for later reading by extracting content and providing a clean reading interface.

### Technical Stack
- **Backend**: PHP 7.4+ (Symfony 5.4)
- **Frontend**: JavaScript (Webpack 5), SCSS/CSS, Material Design
- **Database**: MySQL, PostgreSQL, or SQLite
- **Package Management**: Composer (PHP), Yarn (JavaScript)
- **Build Tools**: Webpack, Babel, PostCSS, Sass
- **Testing**: PHPUnit, DAMA Doctrine Test Bundle
- **Code Quality**: PHP CS Fixer, PHPStan, ESLint, StyleLint, TwigCS

## Project Architecture

### Key Directories
- `src/` - PHP source code (PSR-4 autoloaded as `Wallabag\`)
- `app/` - Symfony application configuration and kernel
- `tests/` - PHPUnit test suite  
- `assets/` - Frontend source files (JS, SCSS)
- `web/` - Web public directory (document root)
- `templates/` - Twig templates
- `migrations/` - Doctrine database migrations
- `fixtures/` - Test data fixtures
- `translations/` - i18n translation files
- `bin/` - Executable scripts (configured via composer)
- `var/` - Cache, logs, sessions (auto-generated)
- `vendor/` - Composer dependencies

### Configuration Files
- `composer.json` - PHP dependencies and autoloading
- `package.json` - JavaScript dependencies and build scripts
- `.php-cs-fixer.dist.php` - PHP code style rules
- `phpstan.neon` - Static analysis configuration
- `phpunit.xml.dist` - Test suite configuration  
- `.eslintrc.json` - JavaScript linting rules
- `stylelint.config.js` - CSS/SCSS linting rules
- `webpack.config.js` - Frontend build configuration
- `app/config/` - Symfony application configuration

## Build and Development Commands

### Environment Setup

**ALWAYS run commands in repository root (`/path/to/wallabag/`)**.

#### Initial Setup
```bash
# Install PHP dependencies (requires composer)
composer install

# Install JavaScript dependencies
yarn install

# Build frontend assets for development
yarn run build:dev

# Build frontend assets for production  
yarn run build:prod
```

#### Using Make Commands
The project provides convenient make targets:
```bash
make help           # Show available commands
make install        # Full installation (prod)
make dev            # Development setup
make build          # Build frontend assets (prod)
make test          # Run test suite
make run           # Start development server
```

### Common Build Issues and Workarounds

#### Composer Installation Issues
If `composer install` fails with GitHub token errors:
- This repository already contains a complete `vendor/` directory from a successful installation
- GitHub token issues typically occur due to API rate limits or specific dependency requirements
- The environment is functional for development even if composer install shows token prompts

**Workaround**: Use existing vendor/ directory. If bin/ executables are missing, they can be recreated by running `composer install --no-scripts` or use system-global versions.

#### Missing bin/phpunit
- PHPUnit executable is typically installed in `vendor/bin/` by composer
- If missing, use system phpunit: `/usr/local/bin/phpunit` or global installation
- **Always set up test database first**: `cp app/config/tests/parameters_test.sqlite.yml app/config/parameters_test.yml`

#### Frontend Build Warnings
- Sass `@import` deprecation warnings are expected (migrating to `@use`)
- Large bundle size warnings for material.js are expected
- "Browserslist outdated" warning can be ignored or fix with `npx update-browserslist-db@latest`

### Testing

#### Running Tests
```bash
# Full test suite (after setting up test DB config)
cp app/config/tests/parameters_test.sqlite.yml app/config/parameters_test.yml
XDEBUG_MODE=off php -dmemory_limit=-1 bin/phpunit -v

# Or using make (preferred)
make test

# Run specific test file
bin/phpunit tests/Controller/EntryControllerTest.php
```

#### Test Database Setup
Tests use SQLite by default. Configuration files available:
- `app/config/tests/parameters_test.sqlite.yml` (recommended)
- `app/config/tests/parameters_test.mysql.yml`
- `app/config/tests/parameters_test.pgsql.yml`

### Linting and Code Quality

#### PHP Code Standards
```bash
# Check code style
bin/php-cs-fixer fix --dry-run --verbose

# Fix code style issues
bin/php-cs-fixer fix

# Static analysis
bin/phpstan analyse

# Composer dependency analysis
bin/composer-dependency-analyser
```

#### Frontend Code Quality
```bash
# ESLint runs automatically during webpack build
yarn run build:dev  # Includes ESLint checking

# Manual ESLint checking
yarn eslint assets/ --ext .js

# StyleLint for CSS/SCSS
yarn stylelint "assets/**/*.scss" --config stylelint.config.js
```

#### Twig Template Quality
```bash
# TwigCS for template validation
bin/twigcs app/ src/ --severity=error
```

### Development Server

```bash
# Symfony built-in server
make run
# or
php bin/console server:run --env=dev

# Access at http://127.0.0.1:8000
```

## GitHub Actions CI/CD

### Workflow Files
- `.github/workflows/continuous-integration.yml` - Main CI (PHPUnit tests)
- `.github/workflows/coding-standards.yml` - Code quality checks
- `.github/workflows/translations.yml` - i18n management

### CI Validation Steps
The CI runs comprehensive checks across multiple PHP versions (7.4-8.3) and databases:

1. **Code Standards** (PHP 7.4):
   - Composer dependency analysis
   - PHP CS Fixer (coding standards)
   - PHPStan (static analysis)  
   - TwigCS (template validation)
   - Composer normalization

2. **Integration Tests** (Multiple PHP/DB combinations):
   - PHPUnit test suite
   - Tests against MySQL, PostgreSQL, SQLite
   - Tests with and without database table prefixes

### Environment Variables
Key environment variables used in CI:
- `DOCTRINE_DEPRECATIONS=none` - Disable Doctrine deprecation notices
- `SYMFONY_ENV` or `APP_ENV=test` - Set environment
- `XDEBUG_MODE=off` - Disable Xdebug for performance

## Docker Support

### Development with Docker
```bash
# Copy environment template
cp docker/php/env.example docker/php/env

# Install dependencies
docker-compose run --rm php composer install
docker-compose run --rm php bin/console wallabag:install
docker-compose run --rm php yarn install  
docker-compose run --rm php yarn build:dev

# Start development stack
docker-compose up -d

# Access at http://127.0.0.1:8000
```

## Common Pitfalls and Best Practices

### Database Configuration
- **Always** use appropriate test database configuration before running tests
- SQLite is the simplest for development and testing
- Database table prefix can be customized but requires consistent configuration

### Cache Management
- Clear Symfony cache after configuration changes: `bin/console cache:clear`
- Frontend assets require rebuild after changes: `yarn run build:dev`
- Test cache is automatically cleared in bootstrap

### Memory and Performance
- Use `-dmemory_limit=-1` for memory-intensive commands (tests, analysis)
- Disable Xdebug (`XDEBUG_MODE=off`) for performance when not debugging

### File Permissions
- Ensure `var/` directory is writable
- Web assets in `web/` must be accessible by web server

## Quick Start for Agents

1. **Verify environment**: Ensure PHP 7.4+, Composer, Node 20+, and Yarn are available
2. **Install dependencies**: `composer install && yarn install`  
3. **Setup test environment**: `cp app/config/tests/parameters_test.sqlite.yml app/config/parameters_test.yml`
4. **Build frontend**: `yarn run build:dev`
5. **Run tests**: `make test` or `XDEBUG_MODE=off php -dmemory_limit=-1 bin/phpunit -v`
6. **Check code quality**: `bin/php-cs-fixer fix --dry-run && bin/phpstan analyse`

**Trust these instructions** - they are based on analysis of the actual codebase, CI configuration, and validated build processes. Only search beyond these instructions if you encounter specific errors not covered here or need to implement features requiring deeper architectural understanding.