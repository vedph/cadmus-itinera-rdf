# Manuscript Mappings

Related resources:

- [events mapping](events.md)
- [links](links.md)
- [JSON mappings](code/manuscript-mappings.json)
- sample events

These mappings refer to _manuscript items_. A _manuscript item_ is an item representing a manuscript. The manuscripts's EID is provided in the item's `MetadataPart`.

## Metadata Part

As for [works](work-mappings.md#metadata-part), we map each manuscript's `eid` metadatum found in its metadata part as the manuscripts's ID. So, the mapping refers to an item's part (the metadata part), rather than directly to the item.

A globally unique URI for the **manuscript** is built as follows:

1. prefix `itn:manuscripts/`;
2. manuscript metadata part's GUID;
3. `/` + manuscript EID (as extracted from the `eid` metadatum).

Here (2) grants the URI's uniqueness, and (1+3) its friendliness.

Also, this node is the subject of a **triple** telling that the manuscript is a CIDOC-CRM `E24_physical_human-made_thing`.

## Historical Events Part

Other than the manuscript itself, Itinera maps its [events](events.md). The general mappings for events apply, using the manuscript-specific thesaurus entries for event types and their related entities.
