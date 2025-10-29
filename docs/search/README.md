# LBWF Search Module

A custom Drupal module that enhances search functionality for the London Borough of Waltham Forest website. This module extends Search API and provides advanced search features, active facets display, and popular search requests.

## ⚠️ NOTE: (TBC)

## Overview

The LBWF Search module provides:

- **Advanced search filters** with content type and topic term facets
- **Active facets display** showing currently applied filters
- **Popular search requests** block for quick access to common searches
- **Custom search form alterations** for the reskin theme
- **Autocomplete integration** with Search API Autocomplete
- **Custom Views field** for displaying grandparent node labels

## Dependencies

- `drupal:views` - For search results display
- `drupal:search_api` - Core search functionality
- `drupal:search_api_autocomplete` - Search autocomplete feature
- `drupal:facets` - For search filtering (soft dependency)

## Components

### Services

#### ActiveFacetsService

**Location:** `src/Service/ActiveFacetsService.php`

**Purpose:** Manages the display and functionality of active search facets.

**Key Features:**

- Tracks active facets from URL query parameters
- Retrieves search result counts from Search API
- Builds "remove" URLs for individual filters
- Generates "clear all" URL to reset filters
- Provides render array for active facets block

**Usage:**

```php
$active_facets_service = \Drupal::service('lbwf_search.active_facets');
$data = $active_facets_service->getActiveFacetsData();
```

**Supported Facets:**

- `content_type` - Content Type filter
- `topic_terms` - Topic filter

#### PopularSearchRequestsService

**Location:** `src/Service/PopularSearchRequestsService.php`

**Purpose:** Provides popular search terms from a taxonomy vocabulary.

**Key Features:**

- Loads terms from `popular_search_requests` vocabulary
- Generates search URLs for each term
- Configurable limit for number of results

**Usage:**

```php
$service = \Drupal::service('lbwf_search.popular_search_requests');
$popular_requests = $service->getPopularRequests(10);
```

### Blocks

#### PopularSearchesBlock

**Location:** `src/Plugin/Block/PopularSearchesBlock.php`

**Block ID:** `lbwf_search_popular_searches`

**Purpose:** Displays popular search terms as clickable links.

**Features:**

- Automatically loads terms from taxonomy
- Caches based on taxonomy term changes
- Links directly to search results

**Usage:**

- Place block via Block Layout UI
- Terms are managed in the "Popular Search Requests" taxonomy vocabulary

### Forms

#### CustomSearchSettings

**Location:** `src/Form/CustomSearchSettings.php`

**Route:** `/admin/config/search/custom-search`

**Purpose:** Configuration form for enabling/disabling advanced search features.

**Settings:**

- **Enable Advanced Search** - Toggle advanced search filters on/off

**State Variable:** `custom_search.enable_advanced_search`

### Custom Views Field

#### GrandparentNodeLabel

**Location:** `src/Plugin/views/field/GrandparentNodeLabel.php`

**Purpose:** Displays the grandparent node's label in Views based on custom hierarchy.

**Usage in Views:**

1. Add field "Grandparent Node Label" to your view
2. Field will automatically traverse node hierarchy
3. Displays the grandparent node's title if available

## Theme Integration

### Custom Templates

The module provides custom Twig templates:

#### Active Facets Block

```twig
{# templates/block--lbwf-active-facets.html.twig #}
- Displays active filters with remove links
- Shows results count
- Includes "Clear all" link
```

#### Search Bar Form

```twig
{# templates/views-exposed-form--generic-search--search-bar-block.html.twig #}
- Custom search form layout
- Search input and button
- Filter toggle button
- Facet filters container
```

#### Popular Search Requests

```twig
{# templates/popular-search-requests.html.twig #}
- List of popular search terms
- Styled as clickable pills/links
```

### Form Alterations

The module significantly alters the `generic_search` view's exposed form for the `lbwf_theme_reskin` theme:

#### Search Bar Enhancements

**Search Input Container:**

- Groups search input and submit button
- Adds placeholder text
- Custom CSS classes for styling

**Search Title:**

- Adds H1 "Search Results" heading
- Positioned above search form

**Advanced Search Toggle** (when enabled):

- "Show/Hide filters" button
- JavaScript-driven toggle functionality
- Accessible with ARIA attributes

**Filters Container:**

- Collapsible container for facets
- Groups `content_type` and `topic_terms` facets
- "Apply filters" button
- "Clear all" link (only shown when filters are active)

**Active Facets Display:**

- Shows currently applied filters
- Individual remove buttons for each filter
- Results count display

### JavaScript Libraries

#### filter-toggle

**Purpose:** Handles show/hide functionality for advanced filters

#### search-filters

**Purpose:** Additional search filter interactions

#### active-facets

**Purpose:** Handles active facets interactions and styling

## Configuration

### Enable Advanced Search

1. Navigate to `/admin/config/search/custom-search`
2. Check "Enable advanced search"
3. Save settings

This will enable:

- Filter toggle button
- Content type facets
- Topic term facets
- Active facets display
- Apply/Clear filter buttons

### Popular Search Requests

1. Navigate to `/admin/structure/taxonomy/manage/popular_search_requests/overview`
2. Add/edit taxonomy terms
3. Each term name becomes a clickable search request
4. Terms automatically appear in the Popular Searches block

## Hooks Implemented

### hook_form_FORM_ID_alter()

Alters the Views exposed form for `generic_search` view:

- Adds custom search elements
- Reorganizes form structure
- Adds active facets display
- Implements filter toggle functionality

**Key Changes:**

- Makes fulltext search required (with core patch)
- Theme-specific alterations for `lbwf_theme_reskin`
- Moves facets into collapsible filters container
- Adds custom buttons and actions

### hook_module_implements_alter()

Ensures `search_api_autocomplete` runs before `lbwf_search` to prevent loss of autocomplete functionality.

### hook_preprocess_views_exposed_form()

Preprocesses the search form template variables:

- Checks for populated facets
- Sets `populated_facets` variable for template

## Technical Notes

### Search API Integration

The module works with Search API views:

- **View ID:** `generic_search`
- **Display ID:** `search_bar_block`
- **Facet Source:** `search_api:views_page__generic_search__search_bar_block`

### Required Core Patch

The module requires a core Drupal patch to make fulltext search fields required without showing errors on page load:

**Patch:** https://www.drupal.org/project/drupal/issues/2568889

**Issue:** `2568889: Views exposed text filter set to required shows an empty error and form error on page load`

Without this patch, required search fields will display validation errors on initial page load.

### Theme Detection

The module checks the active theme and applies specific alterations only for `lbwf_theme_reskin`:

```php
$active_theme = \Drupal::service('theme.manager')->getActiveTheme()->getName();
if ($active_theme === 'lbwf_theme_reskin') {
  // Apply reskin-specific changes
}
```

### URL Query Parameters

The module works with these URL parameters:

- `search` - Search query text
- `content_type` - Content type filter (array)
- `topic_terms` - Topic filter (array)

**Example URL:**

```
/search?search=council%20tax&topic_terms[]=10&content_type[]=page
```

## Development

### Adding New Facets

1. Add facet to `$facet_fields` array in `ActiveFacetsService.php`:

```php
'my_facet' => 'My Facet Label',
```

2. Update form alter to include the new facet:

```php
if (isset($form['my_facet'])) {
  $form['filters']['my_facet'] = $form['my_facet'];
  unset($form['my_facet']);
}
```

3. Add to clearable filters:

```php
$clearable_filter_keys = ['content_type', 'topic_terms', 'my_facet'];
```

### Debugging

The module uses the logger with the 'lbwf_search' log channel.

## Known Issues

TBC

## Maintenance

### When Updating Dependencies

- **Search API updates** - Test fulltext search required functionality
- **Facets module updates** - Verify active facets display
- **Views updates** - Check exposed form alterations

### Testing

Playwright tests cover:

- Search functionality
- Filter application and clearing
- Active facets display
- Popular search requests

See: `/playwright/e2e/specs/search-*.spec.ts`

## Support

For issues or questions:

1. Check Playwright test documentation for known issues
2. Review module logs at `/admin/reports/dblog`
3. Check configuration at `/admin/config/search/custom-search`
4. Verify taxonomy terms at `/admin/structure/taxonomy/manage/popular_search_requests/overview`
