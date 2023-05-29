# Work Mappings

These mappings refer to work items. A work item is an item representing a literary work. The work's EID is provided in the item's `MetadataPart`.

## Item

As each work item has a metadata part, we map its `eid` metadatum as the work's ID. To make it a globally unique URI, the mapping builds it as follows:

1. prefix `itn:works/`
2. work item's GUID
3. `/` + work EID

where (2) grants its uniqueness, and (1+3) its friendliness.

Also, this node is the subject of a triple telling that the work is a CIDOC-CRM E90 symbolic object.

## Events

Events related to the work are listed in the work item's events part. The event type is provided by a thesaurus entry, and so are the relation types for that event with reference to directly related entities.

In general:

- each event corresponds to an entity node, whose URI is `itn:events/PART-GUID/EID`.
- each event has at least 3 triples where the event is the subject:
  - event is an E7 activity.
  - event P2 has type + the event's type as derived from the thesaurus entry. The entity for this entry has URI `itn:event-types/ID`.
  - event P16 used specific object + the work entity. This connects the event with its work.

Also, a number of children mappings are provided for each event mapping. The following mappings are always the same, and thus shared among all the event mappings:

- description for the event's optional description.
- note for the event's optional note.
- chronotopes for the event's optional date(s)/time(s).

Also, there is an additional child mapping for each type of relation in the list of the event's related entities. For instance, the text sent event has these possible relations:

- sender
- recipient
- has participant

Each of these mappings outputs 1 node for the related entity, and at least 1 triple connecting the event with the related entity.

As a sample, this child mapping:

```json
{
  "name": "text sent event/related/has_participant",
  "source": "relatedEntities[?relation=='text:send:recipient']",
  "output": {
    "nodes": {
      "recipient": "{@id.target.gid}"
    },
    "triples": ["{?event} crm:P11_has_participant {?recipient}"]
  }
}
```

matches the relation type `text:send:recipient`, and outputs:

- a node for the recipient with URI derived from the link's target GID.
- a triple telling that event P11 has participant recipient.
