# Person Mappings

Related resources:

- [events mapping](events.md)
- [links](links.md)
- [JSON mappings](code/person-mappings.json)
- sample events

These mappings refer to _person items_. A _person item_ is an item representing a person, usually the author of a work. The persons's EID is provided in the item's `MetadataPart`.

## Metadata Part

As for [works](work-mappings.md#metadata-part), we map each person's `eid` metadatum found in its metadata part as the persons's ID. So, the mapping refers to an item's part (the metadata part), rather than directly to the item.

A globally unique URI for the **person** is built as follows:

1. prefix `itn:persons/`;
2. person metadata part's GUID;
3. `/` + person EID (as extracted from the `eid` metadatum).

Here (2) grants the URI's uniqueness, and (1+3) its friendliness.

Also, this node is the subject of a **triple** telling that the person is a CIDOC-CRM _E21 person_.

## Historical Events Part

Other than the person itself, Itinera maps its [events](events.md). The events connected to a person are the backbone of his/her bio.

The general mappings for events apply, using the person-specific thesaurus entries for event types and their related entities.

## Person Works Part

The works by a person. This is an Itinera-specific part, representing a simple list of work IDs with a conventional title and optional assertion, related to the attribution of the work to the corresponding person item. The part model is:

- `works` (`PersonWork[]`):
  - `eid` (`string`)
  - `title`\* (`string`)
  - `assertion` (`Assertion`)

For each work:

- CREATION `a E65_Creation`;
- `E65_creation` `P94_has_created` WORK,
- `P14_carried_out_by` PERSON.
