# Work Mappings

Related resources:

- [events mapping](events.md)
- [links](links.md)
- [JSON mappings](code/work-mappings.json)
- [sample events](code/work-events.json)

These mappings refer to _work items_. A _work item_ is an item representing a literary work. The work's EID is provided in the item's `MetadataPart`.

## Metadata Part

As each work item has a metadata part, we map its `eid` metadatum as the work's ID. So, the mapping refers to an item's part (the metadata part), rather than directly to the item.

A globally unique URI for the **work** is built as follows:

1. prefix `itn:works/`;
2. work metadata part's GUID;
3. `/` + work EID (as extracted from the `eid` metadatum).

Here (2) grants the URI's uniqueness, and (1+3) its friendliness.

Also, this node is the subject of a **triple** telling that the work is a CIDOC-CRM _E90 symbolic object_.

>Note that here we pick the item's _metadata part_ GUID, rather than the _item_'s GUID itself. Of course, nothing changes for the URI, as the GUID is opaque, and its only purpose is making the URI globally unique. Anyway, the part's GUID here is chosen to be consistent with event's related entities URIs, which refer to parts GUIDs, as nothing ensures that the related entity always corresponds to a whole item, rather then being defined in any of its parts only. Thus, always picking the part's GUID allows consistency, whatever the details of source data modeling, unless of course you are dealing with entities derived from items rather than from any of its parts.

## Historical Events Part

Other than the work itself, Itinera maps its [events](events.md). The general mappings for events apply, using the work-specific thesaurus entries for event types and their related entities.

## Work Info Part

Information about the literary work represented by the item.

- authorIds (AssertedCompositeId[])
  - target (PinLinkTarget):
    - gid\* (string)
    - label\* (string)
    - itemId (string)
    - partId (string)
    - partTypeId (string)
    - roleId (string)
    - name (string)
    - value (string)
  - scope (string)
  - tag (string)
  - assertion (Assertion)
- languages\* (string[]) T:literary-work-languages
- genre\* (string) T:literary-work-genres (hierarchical, single choice)
- metres (string[]) T:literary-work-metres
- strophes (string[])
- isLost (boolean)
- titles\* (AssertedTitle[]):
  - language (string) T:literary-work-languages
  - value (string)
  - assertion (Assertion)
- note (string)

A creation event creates the **work**:

- CREATION `a E65_Creation`;
- CREATION `P94_has_created` WORK.

For each **author**:

- AUTHOR `a E21_Person`;
- CREATION `P14_carried_out_by` AUTHOR.

For each **title**:

- TITLE `a E35_title`;
- TITLE `P190_has_symbolic_content` "TITLE VALUE";
- TITLE `P72_has_language` "language";
- WORK `P102_has_title` TITLE.

If the work is **lost**:

- DESTRUCTION `a E6_Destruction`;
- DESTRUCTION `P13_destroyed` WORK.

TODO The optional **assertion** is mapped like events assertions.
