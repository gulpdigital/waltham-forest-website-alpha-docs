# Entity Share

[Entity Share](https://www.drupal.org/project/entity_share) is a Drupal contrib
module which allows entities to be pulled from one Drupal site into another.

Entities are pulled in to a client site from a server site. In a content
deployment scenario, the client site might be the live site, and the server site
a staging site. A user logs into the client site, and uses an admin UI, where
they select entities to pull. The list of entities shows whether entities on the
server are new, or have updated data.

When an entity is pulled in, it is run through various processors, which handle
things such as pulling in referenced entities and related entities. This means
that pulling in a node will also pull in its paragraphs, media entities, path
alias (but see below for more details), and redirects that point to it.

Pulling an entity which already exists on the client will completely overwrite
its data on the client.

The entity pull admin UI shows multiple channels of entities, each channel for a
single node type. If you are pulling a whole new area of content from a staging
site, you may need to pull entities from more than one channel to get all of the
content into the client site.

## Table of Contents

- [Set-up](../entity-share/entity-share-setup.md)
- [Using](../entity-share/entity-share-using.md)
