# Work Mappings

Related resources:

- [events mapping](events.md)
- [links](links.md)
- [JSON mappings](code/work-mappings.json)
- [sample events](code/work-events.json)

These mappings refer to _work items_. A _work item_ is an item representing a literary work. The work's EID is provided in the item's `MetadataPart`.

## Metadata Part

A metadata part has this schema:

- `metadata` (`Metadatum[]`):
  - `type` (`string`)
  - `name`\* (`string`)
  - `value`\* (`string`)

As each work item has a metadata part, we map its `eid` metadatum as the work's ID. So, the mapping refers to an item's part (the metadata part), rather than directly to the item.

A globally unique URI for the **work** is built as follows:

1. prefix `itn:works/`;
2. work metadata part's GUID;
3. `/` + work EID (as extracted from the `eid` metadatum).

Here (2) grants the URI's uniqueness, and (1+3) its friendliness.

>Note that here we pick the item's _metadata part_ GUID, rather than the _item_'s GUID itself. Of course, nothing changes for the URI, as the GUID is opaque, and its only purpose is making the URI globally unique. Anyway, the part's GUID here is chosen to be consistent with event's related entities URIs, which refer to parts GUIDs, as nothing ensures that the related entity always corresponds to a whole item, rather then being defined in any of its parts only. Thus, always picking the part's GUID allows consistency, whatever the details of source data modeling, unless of course you are dealing with entities derived from items rather than from any of its parts.

Besides the work's node, another node is generated to represent its **creation**; this is because other work item's parts will eventually attach more data to it, like datation and location (chronotopes part) or authors (work info part). So, the creation event is generated once, with a predictable URI; and all the other parts will build triples assuming it as their subject.

The work node is the subject of these **triples**:

- work is a `E90_symbolic_object`;
- event is a `E65_creation`;
- event `crm:p94_has_created` work.

### Metadata Part Example

Say you have these metadata for a work item:

```json
{
  "metadata": [
    {
      "name": "eid",
      "value": "alpha"
    },
    {
      "name": "copyright",
      "value": "(C) some guy 2023"
    }
  ]
}
```

The generated **nodes** are 2, one for the work and another for the event creating it.

>⚠️ To make things clearer, in this example and in all the following ones I replace the mapped part's GUID with `PID`, and the item's metadata part with `MPID`.

| label            | uri                    |
|------------------|------------------------|
| itn:works/alpha  | itn:works/`PID`/alpha  |
| itn:events/alpha | itn:events/`PID`/alpha |

All the nodes have their SID equal to the metadata part's GUID + `/alpha`.

The generated **triples** are 3, telling that the work is an `E90_Symbolic_object`, the event is an `E65_Creation`, and it created that work:

| S                      | P                   | O                       |
|------------------------|---------------------|-------------------------|
| itn:works/`PID`/alpha  | rdf:type            | crm:e90_symbolic_object |
| itn:events/`PID`/alpha | rdf:type            | crm:e65_creation        |
| itn:events/`PID`/alpha | crm:p94_has_created | itn:works/`PID`/alpha   |

As for nodes, all the triples have their SID equal to the metadata part's GUID + `/alpha`.

## Work Chronotopes Part

In Cadmus models, a "chronotope" represents the pairing of a date and a place. Work's chronotopes are represented by a generic chronotopes part. Its model is:

- `chronotopes` (`AssertedChronotope[]`):
  - `place` (`AssertedPlace`)
    - `tag` (`string`)
    - `value` (`string`)
    - `assertion` (`Assertion`):
      - `tag` (`string`)
      - `rank` (`short`)
      - `references` (`DocReference[]`):
        - `type` (`string`)
        - `tag` (`string`)
        - `citation` (`string`)
        - `note` (`string`)
  - `date` (`AssertedDate`):
    - `a`\* (`Datation`):
      - `value`\* (`int`): the numeric value of the point. Its interpretation depends on other points properties: it may represent a year or a century, or a span between two consecutive Gregorian years.
      - `isCentury` (`boolean`): true if value is a century number; false if it's a Gregorian year.
      - `isSpan` (`boolean`): true if the value is the first year of a pair of two consecutive years. This is used for calendars which span across two Gregorian years, e.g. 776/5 BC.
      - `month` (`short`): the month number (1-12) or 0.
      - `day` (`short`): the day number (1-31) or 0.
      - `isApproximate` (`boolean`): true if the point is approximate ("about").
      - `isDubious` (`boolean`): true if the point is dubious ("perhaphs").
      - `hint` (`string`): a short textual hint used to better explain or motivate the datation point.
    - `b` (`Datation`)
    - `tag` (`string`)
    - `assertion` (`Assertion`)

For each chronotope, the mappings may generate a **timespan** node for the datation, and a **place** node for the location; also, other nodes and triples may come from their assertions if any.

### Work Chronotopes Part Example

Say we start from this part, representing two chronotopes alternatives, the first being defined as more probable than the second (via their `rank`):

- (A) place `Arezzo`, date `May 1234`. References:
  - a short bibliographic item (`Rossi 1963`);
  - a manuscript (`Vat.Lat.123`).
- (B) place `Roma`, date `1262`. References:
  - a short bibliographic item (`Verdi 1941`).

```json
{
  "chronotopes": [
    {
      "place": {
        "value": "Arezzo"
      },
      "date": {
        "a": {
          "value": 1234,
          "month": 5
        }
      },
      "assertion": {
        "rank": 1,
        "references": [
          {
            "type": "biblio",
            "citation": "Rossi 1963"
          },
          {
            "type": "ms",
            "citation": "Vat.Lat.123"
          }
        ]
      }
    },
    {
      "place": {
        "value": "Roma"
      },
      "date": {
        "a": {
          "value": 1262
        }
      },
      "assertion": {
        "rank": 2,
        "references": [
          {
            "type": "biblio",
            "citation": "Verdi 1941"
          }
        ]
      }
    }
  ]
}
```

As we have 2 chronotopes, each with both place and date and an assertion, the mapping generates 9 **nodes**: 3 for the place (`Arezzo`), date, and assertion of the first chronotope; and 3 for the place (`Roma`), date and assertion of the second one. Additionally, document references are represented by citations:

| label                | uri                  | sid                       |
|----------------------|----------------------|---------------------------|
| itn:places/arezzo    | itn:places/arezzo    | PID/chronotopes           |
| itn:timespans/ts#52  | itn:timespans/ts#52  | PID/chronotopes           |
| itn:assertions/as#53 | itn:assertions/as#53 | PID/chronotopes/assertion |
| itn:citations/cit#54 | itn:citations/cit#54 | PID/chronotopes/reference |
| itn:citations/cit#55 | itn:citations/cit#55 | PID/chronotopes/reference |
| itn:places/roma      | itn:places/roma      | PID/chronotopes           |
| itn:timespans/ts#56  | itn:timespans/ts#56  | PID/chronotopes           |
| itn:assertions/as#57 | itn:assertions/as#57 | PID/chronotopes/assertion |
| itn:citations/cit#58 | itn:citations/cit#58 | PID/chronotopes/reference |

The generated **triples** are 29: the event which generated the work took place at Arezzo, which is an `E53_Place` at about 1234; or, it took place at Rome, another `E53_Place`, at about 1262.

 | S                       | P                                  | O                            | sid                         |
 |-------------------------|------------------------------------|------------------------------|-----------------------------|
 | itn:places/arezzo       | rdf:type                           | crm:e53_place                | `PID`/chronotopes           |
 | itn:events/`MPID`/alpha | crm:p7_took_place_at               | itn:places/arezzo            | `PID`/chronotopes           |
 | itn:events/`MPID`/alpha | crm:p4_has_time-span               | itn:timespans/ts#52          | `PID`/chronotopes           |
 | itn:timespans/ts#52     | crm:p82_at_some_time_within        | 1234.4166666666667           | `PID`/chronotopes           |
 | itn:timespans/ts#52     | crm:p87_is_identified_by           | May 1234 AD                  | `PID`/chronotopes           |
 | itn:events/`MPID`/alpha | itn:has_probability                | 1                            | `PID`/chronotopes/assertion |
 | itn:assertions/as#53    | rdf:type                           | crm:e13_attribute_assignment | `PID`/chronotopes/assertion |
 | itn:assertions/as#53    | crm:p140_assigned_attribute_to     | itn:events/`MPID`/alpha      | `PID`/chronotopes/assertion |
 | itn:assertions/as#53    | crm:p141_assigned                  | itn:has_probability          | `PID`/chronotopes/assertion |
 | itn:assertions/as#53    | crm:p177_assigned_property_of_type | crm:e55_type                 | `PID`/chronotopes/assertion |
 | itn:citations/cit#54    | rdf:type                           | crm:e31_document             | `PID`/chronotopes/reference |
 | itn:citations/cit#54    | rdfs:label                         | Rossi 1963                   | `PID`/chronotopes/reference |
 | itn:assertions/as#53    | crm:p70i_is_documented_in          | itn:citations/cit#54         | `PID`/chronotopes/reference |
 | itn:citations/cit#55    | rdf:type                           | crm:e31_document             | `PID`/chronotopes/reference |
 | itn:citations/cit#55    | rdfs:label                         | Vat.Lat.123                  | `PID`/chronotopes/reference |
 | itn:assertions/as#53    | crm:p70i_is_documented_in          | itn:citations/cit#55         | `PID`/chronotopes/reference |
 | itn:places/roma         | rdf:type                           | crm:e53_place                | `PID`/chronotopes           |
 | itn:events/`MPID`/alpha | crm:p7_took_place_at               | itn:places/roma              | `PID`/chronotopes           |
 | itn:events/`MPID`/alpha | crm:p4_has_time-span               | itn:timespans/ts#56          | `PID`/chronotopes           |
 | itn:timespans/ts#56     | crm:p82_at_some_time_within        | 1262                         | `PID`/chronotopes           |
 | itn:timespans/ts#56     | crm:p87_is_identified_by           | 1262 AD                      | `PID`/chronotopes           |
 | itn:events/`MPID`/alpha | itn:has_probability                | 2                            | `PID`/chronotopes/assertion |
 | itn:assertions/as#57    | rdf:type                           | crm:e13_attribute_assignment | `PID`/chronotopes/assertion |
 | itn:assertions/as#57    | crm:p140_assigned_attribute_to     | itn:events/`MPID`/alpha      | `PID`/chronotopes/assertion |
 | itn:assertions/as#57    | crm:p141_assigned                  | itn:has_probability          | `PID`/chronotopes/assertion |
 | itn:assertions/as#57    | crm:p177_assigned_property_of_type | crm:e55_type                 | `PID`/chronotopes/assertion |
 | itn:citations/cit#58    | rdf:type                           | crm:e31_document             | `PID`/chronotopes/reference |
 | itn:citations/cit#58    | rdfs:label                         | Verdi 1941                   | `PID`/chronotopes/reference |
 | itn:assertions/as#57    | crm:p70i_is_documented_in          | itn:citations/cit#58         | `PID`/chronotopes/reference |

>Notice that here the URI representing the event which generated the work is provided in the mapping as a metadatum, because it is assumed that the item's mapping is generating it as explained [above](#metadata-part). This is why this event's URI is designed to be predictable. As you can see, the work mapping for `MetadataPart` refers to the part ID (`PID` here for brevity), while here the mapping for `ChronotopesPart` refers to the metadata part ID (`MPID`): so, these two values are the same.

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

For each **author**, we could just say that a CREATION event `P14_carried_out_by` AUTHOR. Yet, this does not allow for assertions about the link between the work and the author. To do this we must reify `P14_carried_out_by` introducing an authorship production event:

- CONFIDENCE_ASSIGNMENT a `E13_Attribute_Assignment` = there is a confidence assignment event which assigns attributes to something; these attributes will consist in the triples expressing our confidence level;
- AUTHORSHIP `a E12_Production`;
- AUTHORSHIP `P14_carried_out_by` AUTHOR;
- AUTHORSHIP `P94_has_created` WORK;
- AUTHORSHIP `P140i_was_attributed_by` CONFIDENCE_ASSIGNMENT.

Now you can attach assertion triples to CONFIDENCE_ASSIGNMENT.

For each **title** we could eventually map it (see [clips](code/clips.json)) this way, but currently we are not interested in them:

- TITLE `a E35_title`;
- TITLE `P190_has_symbolic_content` "TITLE VALUE";
- TITLE `P72_has_language` "language";
- WORK `P102_has_title` TITLE.

If the work is **lost**:

- DESTRUCTION `a E6_Destruction`;
- DESTRUCTION `P13_destroyed` WORK.

Assertions on author identifiers are handled [as usual](named-mappings.md).

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
        "languages": ["en"],
        "assertion": {
          "rank": 1,
          "references": [
            {
              "type": "biblio",
              "citation": "Rossi 1963"
            }
          ]
        }
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

These mappings generate 7 nodes, representing a confidence assignment event, a production event which creates the work, the work itself, a couple of its titles, and a destruction event because this work is marked as lost:

| label                                         | uri                                           | sid                       |
|-----------------------------------------------|-----------------------------------------------|---------------------------|
| itn:assignments/ca#18                         | itn:assignments/ca#18                         | `PID`                     |
| itn:events/production#20                      | itn:events/production#20                      | `PID`                     |
| itn:works/alpha                               | itn:works/`MPID`/alpha                        | `PID`                     |
| <http://www.dbpedia.org/resource/john_milton> | <http://www.dbpedia.org/resource/john_milton> | `PID`                     |
| itn:assertions/as#21                          | itn:assertions/as#21                          | `PID`/assertion           |
| itn:citations/cit#22                          | itn:citations/cit#22                          | `PID`/assertion/reference |
| itn:events/destruction                        | itn:events/destruction#23                     | `PID`                     |

The generated triples are 15:

| S                         | P                                  | O                                             | sid                       |
|---------------------------|------------------------------------|-----------------------------------------------|---------------------------|
| itn:assignments/ca#18     | rdf:type                           | crm:e13_attribute_assignment                  | `PID`                     |
| itn:events/production#20  | rdf:type                           | crm:e12_production                            | `PID`                     |
| itn:events/production#20  | p94_has_created                    | itn:works/`MPID`/alpha                        | `PID`                     |
| itn:events/production#20  | p140i_was_attributed_by            | itn:assignments/ca#18                         | `PID`                     |
| itn:events/production#20  | crm:p14_carried_out_by             | <http://www.dbpedia.org/resource/john_milton> | `PID`                     |
| itn:assignments/ca#18     | itn:has_probability                | 1                                             | `PID`/assertion           |
| itn:assertions/as#21      | rdf:type                           | crm:e13_attribute_assignment                  | `PID`/assertion           |
| itn:assertions/as#21      | crm:p140_assigned_attribute_to     | itn:assignments/ca#18                         | `PID`/assertion           |
| itn:assertions/as#21      | crm:p141_assigned                  | itn:has_probability                           | `PID`/assertion           |
| itn:assertions/as#21      | crm:p177_assigned_property_of_type | crm:e55_type                                  | `PID`/assertion           |
| itn:citations/cit#22      | rdf:type                           | crm:e31_document                              | `PID`/assertion/reference |
| itn:citations/cit#22      | rdfs:label                         | Rossi 1963                                    | `PID`/assertion/reference |
| itn:assertions/as#21      | crm:p70i_is_documented_in          | itn:citations/cit#22                          | `PID`/assertion/reference |
| itn:events/destruction#23 | rdf:type                           | crm:e6_destruction                            | `PID`                     |
| itn:events/destruction#23 | crm:p13_destroyed                  | itn:works/`MPID`/alpha                        | `PID`                     |

These triples say that:

- an attribute assignment event (`itn:assignments/ca#18`) with probability rank 1 (`itn:has_probability`) assigned attributes to the production of a work (its production `p140i_was_attributed_by` the assignment);
- the production event created the work and was carried out by John Milton; also, its probability is defined by an assertion which in turn is documented by some cited references;
- the work has two titles, having two different languages;
- the destruction event destroyed the work (so that now it is lost).

## Historical Events Part

A number of [events](events.md) are related to works. The general mappings for events apply, using the work-specific thesaurus entries for event types and their related entities.

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

| label                                   | uri                                                  |
|-----------------------------------------|------------------------------------------------------|
| itn:works/alpha                         | itn:works/59cdac8e-4152-43c3-9226-36763748cf84/alpha |
| itn:cited-texts/cit                     | itn:cited-texts/cit#11                               |
| <http://www.dbpedia.org/resource/Iliad> | <http://www.dbpedia.org/resource/iliad>              |
| itn:cited-texts/cit                     | itn:cited-texts/cit#12                               |
| itn:cited-texts/cit                     | itn:cited-texts/cit#13                               |

The SID of all the nodes is `87654321-4321-4321-cba9876543210`.

The triples are 11:

| S                      | P                       | O                                                    |
|------------------------|-------------------------|------------------------------------------------------|
| itn:cited-texts/cit#11 | rdf:type                | crm:e33_linguistic_object                            |
| itn:cited-texts/cit#11 | crm:p106i_forms_part_of | <http://www.dbpedia.org/resource/iliad>              |
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

For each ID representing a PERSON:

- APPELLATION a E41_appellation;
- APPELLATION rdfs:label "...";
- ASSIGNMENT a E13_attribute_assignment;
- ASSIGNMENT P14_carried_out_by WORK;
- ASSIGNMENT P141_assigned APPELLATION;
- ASSIGNMENT P140_assigned_attribute_to PERSON;
- PERSON a E21_person;
- PERSON P1_is_identified_by APPELLATION.

### Related Persons Part Example

Say we have this source part, representing 2 persons cited in the work represented by the item, with their pseudonyms and identification:

```js
{
  "persons": [
    {
      "type": "pseudonym",
      "name": "Tityrus",
      "ids": [
        {
          "target": {
            "gid": "http://www.dbpedia.org/resource/Petrarch",
            "label": "Petrarch"
          }
        }
      ]
    },
    {
      "type": "pseudonym",
      "name": "Meliboeus",
      "ids": [
        {
          "target": {
            "gid": "http://www.dbpedia.org/resource/Francesco_Barbato",
            "label": "Francesco Barbato"
          }
        }
      ]
    }
  ]
}
```

The generated nodes are 8, 4 for each person: a name assignment event, a pseudonym, a work related person type derived from a thesaurus, and the target person (here `PID` is a placeholder for this part's GUID):

| label                                               | uri                                                 |
|-----------------------------------------------------|-----------------------------------------------------|
| itn:events/assignment#12                            | itn:events/assignment#12                            |
| itn:names/pseudonym                                 | itn:names/`PID`/pseudonym                           |
| itn:work-rp-types/pseudonym                         | itn:work-rp-types/pseudonym                         |
| <http://www.dbpedia.org/resource/petrarch>          | <http://www.dbpedia.org/resource/petrarch>          |
| itn:events/assignment#13                            | itn:events/assignment#13                            |
| itn:names/pseudonym                                 | itn:names/`PID`/pseudonym                           |
| itn:work-rp-types/pseudonym                         | itn:work-rp-types/pseudonym                         |
| <http://www.dbpedia.org/resource/francesco_barbato> | <http://www.dbpedia.org/resource/francesco_barbato> |

All the nodes have their SID equal to the part GUID.

The triples generated are 18 (`MPID` here is a placeholder for the work item's metadata part GUID):

| S                                                   | P                              | O                                                   |
|-----------------------------------------------------|--------------------------------|-----------------------------------------------------|
| itn:names/`PID`/pseudonym                           | rdf:type                       | crm:e41_appellation                                 |
| itn:names/`PID`/pseudonym                           | rdfs:label                     | Tityrus                                             |
| itn:names/`PID`/pseudonym                           | crm:p2_has_type                | itn:work-rp-types/pseudonym                         |
| itn:events/assignment#14                            | rdf:type                       | crm:e13_attribute_assignment                        |
| itn:events/assignment#14                            | crm:p14_carried_out_by         | itn:works/`MPID`/alpha                              |
| itn:events/assignment#14                            | crm:p141_assigned              | itn:names/`PID`/pseudonym                           |
| itn:events/assignment#14                            | crm:p140_assigned_attribute_to | <http://www.dbpedia.org/resource/petrarch>          |
| <http://www.dbpedia.org/resource/petrarch>          | rdf:type                       | crm:e21_person                                      |
| <http://www.dbpedia.org/resource/petrarch>          | crm:p1_is_identified_by        | itn:names/`PID`/pseudonym                           |
| itn:names/`PID`/pseudonym                           | rdf:type                       | crm:e41_appellation                                 |
| itn:names/`PID`/pseudonym                           | rdfs:label                     | Meliboeus                                           |
| itn:names/`PID`/pseudonym                           | crm:p2_has_type                | itn:work-rp-types/pseudonym                         |
| itn:events/assignment#16                            | rdf:type                       | crm:e13_attribute_assignment                        |
| itn:events/assignment#16                            | crm:p14_carried_out_by         | itn:works/`MPID`/alpha                              |
| itn:events/assignment#16                            | crm:p141_assigned              | itn:names/`PID`/pseudonym                           |
| itn:events/assignment#16                            | crm:p140_assigned_attribute_to | <http://www.dbpedia.org/resource/francesco_barbato> |
| <http://www.dbpedia.org/resource/francesco_barbato> | rdf:type                       | crm:e21_person                                      |
| <http://www.dbpedia.org/resource/francesco_barbato> | crm:p1_is_identified_by        | itn:names/`PID`/pseudonym                           |

For each person, these triples say that:

- the name (here a pseudonym, as specified with `P2_has_type`) is an `E41_Appellation` and has label equal to its value;
- an assignment event is an `E13_Attribute_Assignment`, which has been `P14_carried_out_by` the work, and `P141_assigned` the name to the person;
- the target of the assignment is an `E21_Person`.
