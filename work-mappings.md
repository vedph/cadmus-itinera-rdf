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

- `authorIds` (`AssertedCompositeId[]`)
  - `target` (`PinLinkTarget`):
    - `gid`\* (`string`)
    - `label`\* (`string`)
    - `itemId` (`string`)
    - `partId` (`string`)
    - `partTypeId` (`string`)
    - `roleId` (`string`)
    - `name` (`string`)
    - `value` (`string`)
  - `scope` (`string`)
  - `tag` (`string`)
  - `assertion` (`Assertion`)
- `languages`\* (`string[]`)
- `genre`\* (`string`)
- `metres` (`string[]`)
- `strophes` (`string[]`)
- `isLost` (`boolean`)
- `titles`\* (`AssertedTitle[]`):
  - `language` (`string`)
  - `value` (`string`)
  - `assertion` (`Assertion`)
- `note` (`string`)

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

TODO optional **assertion**

### Work Info Part Example

Say we have this mock part data:

```json
{
  "authorIds": [
    {
      "id": {
        "target": {
          "gid": "http://www.dbpedia.org/resource/John_Milton",
          "label": "John Milton"
        },
        "languages": ["en"]
      }
    }
  ],
  "genre": "epic",
  "metre": "blank verse",
  "isLost": true,
  "titles": [
    {
      "language": "en",
      "value": "Paradise Lost"
    },
    {
      "language": "it",
      "value": "Paradiso perduto"
    }
  ]
}
```

These mappings generate 6 nodes, representing (here `44897701-9354-44aa-8903-86885a98c0f7` is the work item's metadata part ID) a creation event which creates the work, the work itself, a couple of its titles, and a destruction event because this work is marked as lost:

| label                                                | uri                                                  |
|------------------------------------------------------|------------------------------------------------------|
| itn:events/creation                                  | itn:events/creation#32                               |
| itn:works/44897701-9354-44aa-8903-86885a98c0f7/alpha | itn:works/44897701-9354-44aa-8903-86885a98c0f7/alpha |
| http://www.dbpedia.org/resource/john_milton          | http://www.dbpedia.org/resource/john_milton          |
| itn:titles/title#33                                  | itn:titles/title#33                                  |
| itn:titles/title#34                                  | itn:titles/title#34                                  |
| itn:events/destruction                               | itn:events/destruction#35                            |

The SID for all the nodes is `87654321-4321-4321-cba9876543210`.

The generated triples are 13:

| S                                                    | P                             | O                                                    |
|------------------------------------------------------|-------------------------------|------------------------------------------------------|
| itn:events/creation#32                               | rdf:type                      | crm:e65_creation                                     |
| itn:events/creation#32                               | crm:p94_has_created           | itn:works/44897701-9354-44aa-8903-86885a98c0f7/alpha |
| itn:events/creation#32                               | crm:p14_carried_out_by        | http://www.dbpedia.org/resource/john_milton          |
| itn:works/44897701-9354-44aa-8903-86885a98c0f7/alpha | crm:p102_has_title            | itn:titles/title#33                                  |
| itn:titles/title#33                                  | rdf:type                      | crm:e35_title                                        |
| itn:titles/title#33                                  | crm:p190_has_symbolic_content | Paradise Lost                                        |
| itn:titles/title#33                                  | p72_has_language              | en                                                   |
| itn:works/44897701-9354-44aa-8903-86885a98c0f7/alpha | crm:p102_has_title            | itn:titles/title#34                                  |
| itn:titles/title#34                                  | rdf:type                      | crm:e35_title                                        |
| itn:titles/title#34                                  | crm:p190_has_symbolic_content | Paradiso perduto                                     |
| itn:titles/title#34                                  | p72_has_language              | it                                                   |
| itn:events/destruction#35                            | rdf:type                      | crm:e6_destruction                                   |
| itn:events/destruction#35                            | crm:p13_destroyed             | itn:works/44897701-9354-44aa-8903-86885a98c0f7/alpha |

The SID for all the triples is `87654321-4321-4321-cba9876543210`.

These triples say that:

- the creation event created the work and was carried out by John Milton;
- the work has two titles, having two different languages;
- the destruction event destroyed the work (so that now it is lost).

## Referenced Texts Part

Special part about texts referenced by the item's text. This is a list of all the item's text relevant passages which explicitly or implicitly refer to another text.

The source part's model is:

- `texts` (`ReferencedText[]`):
  - `type`\* (`string`)
  - `targetId`\* (`AssertedCompositeId`)
  - `targetCitation` (`string`)
  - `sourceCitations` (`string[]`)
  - `assertion` (`Assertion`)

The work item represents the referencing text; target ID identifies the referenced text.

In general, to say that work A's chapter 10 is referred to bywork B's chapter 3, we can use `P67i_is_referred_to_by` to link the chapters as instances of `E33_Linguistic_Object`, and use `P3_has_note` to document their chapter numbers as literals.

So in the end you for each textual link will create an entity for the referencing text (as `E33_Linguistic_Object`) and a note for its citation, e.g.:

- WORK_A_CHAPTER_10 `a E33_Linguistic_Object`;
- WORK_A_CHAPTER_10 `P106i_forms_part_of` WORK_A;
- WORK_A_CHAPTER_10 `P3_has_note` "Chapter 10";
- WORK_A_CHAPTER_10 `P67i_is_referred_to_by` WORK_B_CHAPTER_3;

- WORK_B_CHAPTER_3 a `E33_Linguistic_Object`;
- WORK_A_CHAPTER_3 `P106i_forms_part_of` WORK_B;
- WORK_B_CHAPTER_3 `P3_has_note` "Chapter 3".

This means that work A's chapter 10 is a propositional object that is referred to by work B's chapter 3, and both chapters have notes with literals that indicate their chapter numbers.

>Alternatively, you could use `P148_has_component` to link the chapters as instances of `E33_Linguistic_Object`, and use `P149_is_identified_by` to link them to instances of `E75_Conceptual_Object_Appellation` that have literals as their values. For example, you could output something like:

```txt
- WORK_A_CHAPTER_10 a E33_Linguistic_Object;
- WORK_A_CHAPTER_10 P148_has_component WORK_B_CHAPTER_3;
- WORK_A_CHAPTER_10 P149_is_identified_by WORK_A_CHAPTER_10_NUMBER;

- WORK_B_CHAPTER_3 a E33_Linguistic_Object;
- WORK_B_CHAPTER_3 P149_is_identified_by WORK_B_CHAPTER_3_NUMBER;

- WORK_A_CHAPTER_10_NUMBER a E75_Conceptual_Object_Appellation;
- WORK_A_CHAPTER_10_NUMBER rdfs:label "Chapter 10";

- WORK_B_CHAPTER_3_NUMBER a E75_Conceptual_Object_Appellation;
- WORK_B_CHAPTER_3_NUMBER rdfs:label "Chapter 3".
```

>This means that work A's chapter 10 is a propositional object that has component work B's chapter 3, and both chapters are identified by conceptual object appellations that have literals as their labels.

So, here for each referenced text we enumerate each of its source citations. By focusing on source citations, we can handle pairs of citations, each related to a work.

### Referenced Texts Part Example

Say we have this mock part data, being the part of a work item:

```json
{
  "texts": [
    {
      "type": "imitation",
      "targetId": {
        "target": {
          "gid": "@http://www.dbpedia.org/resource/Iliad",
          "label": "Iliad"
        }
      },
      "targetCitation": "1.1",
      "sourceCitations": [
        "12.34",
        "56.78"
      ]
    }
  ]
}
```

This part tells us that the work represented by our item (whose metadata part GUID is `59cdac8e-4152-43c3-9226-36763748cf84`) has two passages (12.34 and 56.78) referring to passage `1.1` of another work, the _Iliad_, here identified by an external global ID from DBPedia (and as such, conventionally prefixed by `@`).

The [mappings](code/work-mappings.json) generate 5 nodes: one for the source work, one for the target work (_Iliad_), and 3 citations:

| label                                 | uri                                                  |
|---------------------------------------|------------------------------------------------------|
| itn:works/alpha                       | itn:works/59cdac8e-4152-43c3-9226-36763748cf84/alpha |
| itn:cited-texts/cit                   | itn:cited-texts/cit#11                               |
| http://www.dbpedia.org/resource/Iliad | http://www.dbpedia.org/resource/iliad                |
| itn:cited-texts/cit                   | itn:cited-texts/cit#12                               |
| itn:cited-texts/cit                   | itn:cited-texts/cit#13                               |

The SID of all the nodes is `87654321-4321-4321-cba9876543210`.

The triples are 11:

| S                      | P                       | O                                                    |
|------------------------|-------------------------|------------------------------------------------------|
| itn:cited-texts/cit#11 | rdf:type                | crm:e33_linguistic_object                            |
| itn:cited-texts/cit#11 | crm:p106i_forms_part_of | http://www.dbpedia.org/resource/iliad                |
| itn:cited-texts/cit#11 | crm:p3_has_note         | 1.1                                                  |
| itn:cited-texts/cit#12 | rdf:type                | crm:e33_linguistic_object                            |
| itn:cited-texts/cit#12 | crm:p106i_forms_part_of | itn:works/59cdac8e-4152-43c3-9226-36763748cf84/alpha |
| itn:cited-texts/cit#12 | crm:p3_has_note         | 12.34                                                |
| itn:cited-texts/cit#12 | p67_refers_to           | itn:cited-texts/cit#11                               |
| itn:cited-texts/cit#13 | rdf:type                | crm:e33_linguistic_object                            |
| itn:cited-texts/cit#13 | crm:p106i_forms_part_of | itn:works/59cdac8e-4152-43c3-9226-36763748cf84/alpha |
| itn:cited-texts/cit#13 | crm:p3_has_note         | 56.78                                                |
| itn:cited-texts/cit#13 | p67_refers_to           | itn:cited-texts/cit#11                               |

The SID of all the triples is `87654321-4321-4321-cba9876543210`. These triples say that:

- the target citation (an `E33_linguistic_object`) is 1.1 from work _Iliad_;
- each of the two source citations (other `E33_linguistic_object`'s) is from the work item, and they refer to its passages `12.34` and `56.78` respectively.

## Related Persons Part

Textual labels referencing a person to be identified:

- `persons` (`RelatedPerson[]`):
  - `type`\* (`string`)
  - `name`\* (`string`)
  - `ids`\* (`AssertedCompositeId[]`)
