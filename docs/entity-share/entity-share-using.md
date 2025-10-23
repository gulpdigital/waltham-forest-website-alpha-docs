# Entity Share usage

Entity Share lets you copy entities from one environment to another.

- The environment they are coming from is called the server.
- The environment they are going into is called the client.
- Copying entities across is usually referred to as ‘doing an entity pull'.

Typically, entities will be copied from a staging environment into a merge, a
production, or a deployment environment.

## Special considerations for content editors

Because of how Drupal core handles path aliases, special case is needed when
changing them so that they are pulled correctly with Entity Share.

If the intention is to change an existing alias (for example,
'/content/my-page') to point to a new node, then the path alias must NOT be
entered in the edit form for the new node. Rather, the path_alias entity itself
must be edited in the path alias admin UI to point to the new node. The steps
are:

1. Create the new node, but do NOT give it a path alias. Most node types are set
   up to make an alias automatically, so you will need to deselect that option
   in the node edit form.
2. Go to  Administration › Configuration › Search and metadata › URL aliases
3. Use the filter to find the alias in question, which currently points to the
   old node
4. Click the edit button
5. Make a note of the old node’s path
6. Change the system path for the alias to point to the new node, e.g.
   '/node/12345'
7. Save the alias
8. Go to Administration › Configuration › Search and metadata › URL Redirects
9. Click ‘Add redirect’
10. Enter the old node’s path as the ‘Path’ value
11. Enter the new node’s internal path as the ‘To’ value, e.g. ‘/node/12345’
12. Save the redirect.

When the new node is pulled, the alias on the server will be updated to point to
the new node.

## Pulling entities

1. Ensure the client site has a recent backup.
2. Go to the **client site**, that is the site that will **receive** the new
   content.
3. Go to Admin > Content > Entity Share > Pull entities (or
   `/admin/content/entity_share/pull`)
4. In the Pull entities form, select:
   1. The Import configuration. This should be ‘Gulp content staging’ unless you
      are importing the homepage.
   2. The Remote website.
   3. When you have selected both of those, the Channel dropdown appears.
   4. Select the channel for type of content you want to pull. There are
      channels for various node types, menus, content blocks, and terms.
   5. The list of entities for the channel appears. You can page through this,
      or search the titles to find the entities you need.
   6. Select the entities you want.
5. Click Synchronize entities. This starts a batch process which pulls the
   selected entities and any dependencies.
6. If there were any errors, look at the 'Import statuses' page and the site log
   for details.

## Deploying content from Gulp content-staging

For deploying Gulp's content, entities will be copied

- from `content-staging` https://content-staging-y3udfma-m4rfedipzmuta.eu-5.platformsh.site
- into `content-merge-staging` https://content-merge-staging-iwifpky-m4rfedipzmuta.eu-5.platformsh.site

# Deployment process

1. Get the live DB
2. Merge the code from `develop` into `develop-v2`.
3. Empty the DB on merge: `platform drush -e content-merge-staging sql-drop`. We
   need to empty the DB first, as otherwise tables like `entity_import_status`
   that aren’t on live will persist and cause errors
4. Import the DB into merge: `platform sql --relationship database -e content-merge-staging < DATABASE`
5. Change the admin password if necessaru: `platform drush -e content-merge-staging upwd lbwf_digital PASSWORD`
6. Config import: `platform drush -e content-merge-staging cim -y`
7. You might need to repeat step 4 until there are no errors!
8. Even if there are no config import errors, do a few more config imports for
   luck – some things (such as the homepage search block) have complex
   dependencies and need a bit of a kick to show up properly). Note however that
   config will never show as completely synchronized, because of the `edit_uuid`
   module adding to all the entity displays. If the config import only shows it
   updated entity display entities, then it’s done.
9. Enter the user details for the content-staging environment at /admin/config/services/entity_share/remote.
10. Check the cron status of the client site. If cron is going to run soon,
    disable it so it doesn’t run during the pull operation.
11. Go to entity pull page. If you see error 'Will not follow more than 5
    redirects', then you need to create the ES user on the content-staging
    environment.
    1. Go to /admin/people/create.
    2. Enter the user details to match those entered on the client.
    3. Enter role 'Content deployer'
12. Pull the relevant channels:
    1. Term - Popular Search Requests: whole channel
    2. Block content - Mini Banner: the block with the weird numeric name (this will change as it’s the entity ID)
    3. Block content - Text Block: the ‘Footer - Accessibility’ block
    4. Content - Service landing page: the following nodes:
       1. ‘Council tax’
       2. ‘Schools, education and learning’
       3. ‘Housing’ - NOPE! details to come!
    5. Menu - Megamenu menus: whole channel
    6. Menu - Main navigation: whole channel
    7. Menu - Footer - Popular tasks: whole channel
    8. Menu - Footer - Contact: whole channel
    9. Content - New homepage: pull the ‘Homepage V2' node. **WARNING** Change
       the config to the one specific for the homepage, AND pull this AFTER the
       sections, as it links to some new pages in those.
    10. Redirects
