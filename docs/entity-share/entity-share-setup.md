## Configuration

# Set-up

Entity Share's configuration setup is fairly complex, and requires the following
multiple modules and elements:

- on the client:
  - the Entity Share Client module and dependencies
  - an import configuration config entity
  - a remote site config entity
  - a role for users which allows accessing the Entity Share pull form
- on the server:
  - the Entity Share Server module and dependencies
  - modules which grant necessary access to entity types such as path_alias
  - an entity share channel config entity for each entity bundle that should be
    shown on the client's Entity Share pull form
  - a role for the user which logs into the server site to retrieve entities
    during the pull process.
  - a user account with this role

The mix of config on the server and the client can get a bit confusing: the user
doing the entity pull is logged in on the client, and uses the import
configuration (on the client) and the remote site (on the client) to get a list
of channels (fetched from the server).

The server configuration is in a config split, `entity_share_content_staging`,
so that only sites that need to act as servers have that access; whereas the
client configuration is in main site config, so any site can pull.

Further details are below.

### Client site

#### Import configuration

The Import Configuration is a config entity which specifies the processors to
use on pulled entities. This is a similar architecture to SearchAPI: multiple
processors can be applied at multiple stages in the lifecycle of an entity being
pulled.

We have several Import Configurations for various purposes:

- gulp_content_staging: The general Import Configuration.
- gulp_content_staging_homepage: Specific to the homepage; does not use the path
  alias component filter (see below).
- gulp_dev_staging_files_media: Probably no longer needed; was used to import
  missing files.

Of note, we are using:

- Path alias processor, which queries the server for a path_alias entity which
  points to the pulled node. Because the path_alias field on entities isn't a
  true reference field, these have to be queried for by the alias value, rather
  than by path_alias entity ID, which means special care is needed when updating
  an alias: see the section above.
- Entity reference processor, which pulls in all referenced entities,
  recursively.
- File processor, which retrieves the physical files for file entities.
- Link internal content processor. This handles following link fields whose URIs
  are internal (e.g. ‘internal:node/1234’), and updating them for local IDs. For
  this to work, JSON:API Extras module needs to be present, and to have the
  ‘UUID for link (link field only) (Entity Share)' field enhancer enabled on
  both client and server websites. This is a bit fiddly to set up in the UI! The
  bundle that has the link field needs to be set to 'overwrite’, and then the
  enhancer needs to be set on the link field in question.
- Path alias component filter: Prevents the various processor which pull related
  entities (such as the Entity Reference processor, and the Link processor) from
  pulling entities whose paths place them outside of the current entity's path.
  This means that nodes can be pulled as sections, and not pull content from
  other parts of the site. For example, if the node with path /schools/primary
  is pulled, then a link in that node to /schools/primary/applying will be
  pulled, but not /schools/secondary, or /council-tax.
- Revision author: Sets the revision author if the user exists on the client.
  Does not pull user entities. This requires user UUIDs to be the same on both
  environments.

We are not using the Redirect processor as that causes problems; rather,
redirects need to be pulled separately after content has been pulled.

#### Remote site configuration

The Remote Site is a config entity which holds details for a server site: its
URL and its authorisation details.

For Waltham Forest, we are using the Basic Auth authorisation plugin, which
requires the server to have the core 'Basic Auth' module enabled. User account
name and password are stored in the key_value table, so that they are not in
config. (If Key module is enabled, they are stored with that instead.)

### Server site

#### User account and role

The server site needs a user account who will access the site to retrieve
JSON:API data for pulled entities. This is the user account whose details are in
the Remote Site config entity on the _client_ site.

For Waltham Forest, we have a dedicated user role, `content_deployer`, with a
very limited set of permissions, for security. In addition to the permissions
for authenticated users, this role has the following permissions:

- entity_share_server_access_channels: this is necessary so that the entity
  pull form on the client can fetch the list of channels.
- access path_aliases: this comes from a contrib module, as path_alias
  entities are not accessible by default and the core module only provides an
  admin permission.
- access menu_link_content entities: this comes from a contrib module, as menu
  items are not accessible by default, and core only provides an admin
  permission
- view redirects: this comes from a patch to the redirect module, as currently
  this only provides an admin permission for redirect entities.
- editorial transition permissions: this is needed so the user can see the
  revision_log field.
- view all revisions: this is needed so the user can see the revision_log field.
- view latest version and permissions for unpublished nodes and paragraphs:
  these allow the user to pull an archived node.

#### Entity Share Channels

The Channel entity specifies a way of getting a list of entities from the
server. When the user on the client site select an import configuration and a
remote site, the form fetches the list of channels from the server.

At its simplest, a channel specifies an entity type and a bundle. Channels can't
mix bundles, because Entity Share is tied closely to JSON:API, and Drupal's
implementation of JSON:API has separate endpoints for each bundle.

There are channels for the following in the server config split:

- node types
- block content types
- files
- media types
- taxonomy term vocabularies
- menus, using filters so each channel is one menu
- redirects

## Logging

When an entity is pulled for the first time, Entity Share creates an Import
Status entity for it. This tracks the remote site, channel ID, and last import
date.
