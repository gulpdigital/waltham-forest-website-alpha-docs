# London Borough of Waltham Forest Website

A Drupal 10 website built with LocalGov Drupal distribution and a custom reskin theme powered by Vite and Storybook.

## Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Development](#development)
  - [Theme Development](#theme-development)
  - [Available Scripts](#available-scripts)
  - [Linting and Code Quality](#linting-and-code-quality)
  - [Search Filters](docs/search/README.md)
- [Project Structure](#project-structure)
- [Testing](#testing)
- [Deployment](#deployment)
- [Entity share](docs/entity-share/readme.md)
- [Environment Configuration](#environment-configuration)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## Overview

The London Borough of Waltham Forest public website is built on:

- **Drupal 10** with the LocalGov Drupal distribution
- **Custom Reskin Theme** (`lbwf_theme_reskin`) with modern frontend tooling
- **DDEV** for local development environment
- **Platform.sh** for hosting and deployment

## Tech Stack

### Backend

- **Drupal 10.2+** - Content Management System
- **LocalGov Drupal** - Local government-focused Drupal distribution
- **PHP 8.3** - Server-side language
- **MariaDB 10.11** - Database
- **Solr** - Search engine
- **Memcached** - Caching layer

### Frontend & Build Tools

- **Node.js 18+** - JavaScript runtime
- **Vite** - Modern build tool and dev server
- **Sass** - CSS preprocessor
- **Storybook 8.5+** - Component development environment
- **PostCSS & Autoprefixer** - CSS processing

### Development Tools

- **DDEV** - Local development environment
- **ESLint** - JavaScript linting
- **Stylelint** - CSS linting
- **Prettier** - Code formatting
- **Husky** - Git hooks
- **Nightwatch** - End-to-end testing
- **Playwright** - End-to-end testing and accessibility testing

## Prerequisites

### Required Software

- **DDEV** - [Installation guide](https://ddev.readthedocs.io/en/stable/users/install/)
  - Ensure you have a compatible Docker version (check DDEV docs for latest compatibility)
- **Docker** - Required by DDEV
- **Node.js 18+** - For theme development
- **Composer** - PHP dependency manager (included with DDEV)

### System Requirements

- **macOS, Linux, or Windows** with WSL2
- **4GB+ RAM** recommended for local development
- **20GB+ disk space** for database and files

## Quick Start

1. **Clone the repository:**

   ```bash
   git clone https://github.com/gulpdigital/waltham-forest-current-website.git
   cd waltham-forest-website
   ```

2. **Start DDEV environment:**

   ```bash
   ddev start
   ```

3. **Install PHP dependencies:**

   ```bash
   ddev composer install
   ```

4. **Install and build theme assets:**

   ```bash
   # Root-level dependencies (for testing and linting)
   ddev npm install

   # Theme dependencies and build
   ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm install && npm run build"
   ```

5. **Import database or install Drupal:**
   - **Fresh install:** Visit your DDEV URL and follow Drupal installation (choose LocalGov profile)
   - **Import existing database:** Place your database dump in the project root and run:
     ```bash
     ddev import-db --src=your-database.sql.gz
     ddev drush cr
     ```

6. **Access your site:**
   - **Website (Local):** https://waltham-forest-current-website.ddev.site
   - **Mailpit (email):** https://waltham-forest-current-website.ddev.site:8026
   - **demo (Platform):** 🔗 [demo environment](https://demo-fklvc3a-m4rfedipzmuta.eu-5.platformsh.site/) Used by testers to verify new features and fixes before promotion.
   - **content-staging (Platform):** 🔗 [content-staging environment](https://content-staging-y3udfma-m4rfedipzmuta.eu-5.platformsh.site/) Holds the most up-to-date content. Used by content editors to review content in situ in the new theme.
   - **content-merge-staging (Platform):** 🔗 [content-merge-staging environment](https://content-merge-staging-iwifpky-m4rfedipzmuta.eu-5.platformsh.site/) Used by developers to pull in the latest content via Entity Share for integration checks. ⚠️ DO NOT edit this environment, as it might get wiped.
   - **qa-branch (Platform):** 🔗 [qa-branch environment](https://qa-branch-oc75l6y-m4rfedipzmuta.eu-5.platformsh.site/) Used by testers to verify new features and fixes before promotion.

## Development

### Theme Development

The custom theme (`lbwf_theme_reskin`) uses modern frontend tooling:

#### Development Mode

```bash
# Start theme development with live reloading
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run watch"
```

This starts:

- **Vite build watcher** - Compiles and watches for changes
- **Stylelint watcher** - CSS linting on file changes
- **ESLint watcher** - JavaScript linting on file changes

#### Vite Development Server

```bash
# Start Vite dev server with HMR
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run vite:dev"
```

Access at: https://waltham-forest-current-website.ddev.site:5173

#### Storybook Component Development

```bash
# Build and serve Storybook
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run storybook:build && npm run storybook:dev"
```

Access at: https://waltham-forest-current-website.ddev.site:6006

### Available Scripts

#### Root Level Scripts

```bash
# Testing
ddev npm run test              # Run Nightwatch tests

# Code Quality
ddev npm run phpcbf           # Fix PHP code style issues
```

#### Theme Scripts

```bash
# Build commands
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run build"     # Production build
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run watch"     # Development build + watch

# Vite commands
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run vite:build"    # Build assets
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run vite:watch"    # Watch and rebuild
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run vite:dev"      # Dev server with HMR

# Storybook commands
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run storybook:dev"   # Start Storybook dev server
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run storybook:build" # Build static Storybook

# Linting and fixing
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run eslint"         # Check JS/TS files
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run eslint:fix"     # Fix JS/TS issues
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run stylelint"      # Check CSS/SCSS files
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run stylelint:fix"  # Fix CSS/SCSS issues
```

### Linting and Code Quality

The project uses pre-commit hooks to maintain code quality:

#### Automatic Linting (via Husky/lint-staged)

- **JavaScript/TypeScript** files are checked with ESLint and formatted with Prettier
- **CSS/SCSS** files are checked with Stylelint and formatted with Prettier
- **PHP** files are checked with PHP_CodeSniffer

#### Manual Linting

```bash
# JavaScript/TypeScript
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run eslint"
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run eslint:fix"

# CSS/SCSS
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run stylelint"
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run stylelint:fix"

# PHP
ddev composer phpcs        # Check PHP code standards
ddev composer phpcbf       # Fix PHP code standard violations
```

## Project Structure

```
├── .ddev/                          # DDEV configuration
├── .platform.app.yaml             # Platform.sh configuration
├── assets/                         # Build assets and configurations
├── composer.json                   # PHP dependencies
├── package.json                    # Node.js dependencies (root level)
├── patches/                        # Drupal core and contrib patches
├── web/                           # Drupal web root
│   ├── modules/custom/            # Custom Drupal modules
│   ├── themes/custom/
│   │   └── lbwf_theme_reskin/     # Main custom theme
│   │       ├── components/        # Component files
│   │       ├── src/              # Source SCSS/JS files
│   │       ├── package.json      # Theme-specific dependencies
│   │       ├── vite.config.js    # Vite configuration
│   │       └── .storybook/       # Storybook configuration
│   └── sites/default/            # Drupal site configuration
├── tests/                         # Test configurations
├── scripts/                       # Build and deployment scripts
└── README.md                     # This file
```

## Testing

The project uses two testing frameworks:

### Nightwatch (End-to-End Testing)

Nightwatch tests are configured at the root level:

```bash
# Run all Nightwatch tests
ddev npm run test

# Run with specific retry settings for flaky tests
ddev npm run test -- --retry 2
```

### Playwright (E2E & Accessibility Testing)

Playwright provides comprehensive end-to-end and accessibility testing. Tests are located in the `/playwright` directory.

#### Test Structure

```
playwright/
├── e2e/
│   ├── specs/                    # Test specifications
│   │   ├── accessibility.spec.ts # Accessibility tests with axe-core
│   │   ├── homepage.spec.ts      # Homepage functionality tests
│   │   ├── search-interactions.spec.ts
│   │   └── search-results-interactions.spec.ts
│   ├── page-objects/             # Reusable page object classes
│   └── utils/                    # Test utilities and helpers
├── playwright.config.ts          # Playwright configuration
└── package.json
```

#### Running Playwright Tests

**Prerequisites:**

```bash
cd playwright
npm install
```

**Run all tests:**

```bash
cd playwright
npm run test
```

**Run tests in UI mode** (interactive debugging):

```bash
cd playwright
npm run ui
```

**Run specific test suites:**

```bash
# Accessibility tests only
cd playwright
npm run test -- --grep @accessibility

# Search tests only
cd playwright
npm run test -- --grep @search

# Specific test file
cd playwright
npm run test e2e/specs/homepage.spec.ts
```

#### Test Suites

**1. Accessibility Tests** (`accessibility.spec.ts`)

- Validates WCAG compliance using `@axe-core/playwright`
- Tests homepage, search results, task pages, service pages, and event pages
- Generates detailed violation reports
- Currently allows 1 known violation per page (being tracked)

**2. Homepage Tests** (`homepage.spec.ts`)

- Validates page title and core homepage elements
- Ensures homepage loads correctly

**3. Search Interactions** (`search-interactions.spec.ts`)

- Tests search block visibility and functionality
- Validates search with valid and invalid queries
- Tests "no results" messaging
- Verifies search result counts (e.g., "pcn" query expects 24 results)
- Tests service-specific filtering (some tests skipped pending fixes)

**4. Search Results Interactions** (`search-results-interactions.spec.ts`)

- Tests search results page with various filter combinations
- Validates topic filter application and clearing
- Tests active filter display
- Includes notes on known bugs being tracked

#### Viewing Test Results

After running tests, view the HTML report:

```bash
# Report location
playwright/playwright-report/index.html

# Open in browser (macOS)
open playwright/playwright-report/index.html
```

The report includes:

- Test pass/fail status with execution times
- Screenshots and traces for failed tests
- Accessibility violation details with recommendations

#### Running in Docker/Podman

```bash
# Using Docker
cd playwright
docker compose build
docker compose run playwright npm run test

# Using Podman
cd playwright
podman compose build
podman compose run playwright npm run test
```

#### Debugging Tests

```bash
# Run with visible browser
cd playwright
npm run test -- --headed

# Debug mode (step through tests)
cd playwright
npm run test -- --debug

# UI mode (best for debugging)
cd playwright
npm run ui
```

#### Playwright Configuration

- **Browser:** Chromium (Desktop Chrome)
- **Parallel execution:** Enabled
- **Retries:** 2 on CI, 0 locally
- **Reporter:** HTML
- **Trace:** Captured on first retry

### Manual Testing

- Test responsive design across different screen sizes
- Manual accessibility testing with screen readers
- Cross-browser compatibility verification
- Form validation and error handling

## Deployment

### Platform.sh Deployment

The site is deployed to Platform.sh with automatic builds:

1. **Push to main branch** triggers automatic deployment
2. **Build process** runs:
   - `composer install` for PHP dependencies
   - `npm ci` and `npm run build` for theme assets
   - Database updates and cache clearing

### Manual Deployment Commands

```bash
# Build theme for production
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run build"

# Clear Drupal caches
ddev drush cr

# Update database
ddev drush updb -y

# Import configuration changes
ddev drush cim -y

# export configuration changes
ddev drush cex -y
```

## Environment Configuration

### Development (DDEV)

- **URL:** https://waltham-forest-current-website.ddev.site
- **Database:** MariaDB 10.11
- **PHP:** 8.3
- **Node.js:** Latest
- **Xdebug:** Available (`ddev xdebug on`)

### Environment Variables

Key environment variables are configured in:

- `.ddev/config.yaml` - DDEV-specific settings
- `.platform.app.yaml` - Platform.sh configuration
- `web/sites/default/settings.php` - Drupal settings

## Troubleshooting

### Common Issues

#### DDEV Won't Start

```bash
ddev stop
ddev start
# If issues persist:
ddev restart
```

#### Theme Assets Not Building

```bash
# Clear npm cache and reinstall
ddev exec "cd web/themes/custom/lbwf_theme_reskin && rm -rf node_modules package-lock.json && npm install"

# Rebuild assets
ddev exec "cd web/themes/custom/lbwf_theme_reskin && npm run build"
```

#### Database Connection Issues

```bash
# Check DDEV status
ddev describe

# Restart database container
ddev restart
```

#### Permission Issues

```bash
# Fix file permissions
ddev exec "chown -R www-data:www-data web/sites/default/files"
```

### Development Tips

1. **Use Vite dev server** for faster development with hot module replacement
2. **Leverage Storybook** for isolated component development
3. **Run linters** before committing to catch issues early
4. **Use DDEV commands** for consistency across team members
5. **Check logs** with `ddev logs` if something isn't working

### Getting Help

- **DDEV Documentation:** https://ddev.readthedocs.io/
- **LocalGov Drupal:** https://github.com/localgovdrupal/localgov
- **Drupal Documentation:** https://www.drupal.org/docs
- **Vite Documentation:** https://vitejs.dev/
- **Storybook Documentation:** https://storybook.js.org/

## Contributing

1. Create a feature branch from `main`
2. Make your changes following the coding standards
3. Run linting and tests: `npm run test`
4. Commit with descriptive messages
5. Push and create a pull request

### Coding Standards

- **PHP:** Follow Drupal coding standards (enforced by PHP_CodeSniffer)
- **JavaScript:** ESLint with Prettier formatting
- **CSS/SCSS:** Stylelint with BEM methodology where applicable
- **Git:** Use conventional commit messages
