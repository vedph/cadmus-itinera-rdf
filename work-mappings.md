# Work Mappings

- [Work Mappings](#work-mappings)
  - [Item](#item)
  - [Events](#events)
    - [Event Assertions](#event-assertions)
    - [Groups of Events](#groups-of-events)
    - [General Conventions](#general-conventions)
    - [Itinera Conventions](#itinera-conventions)

Related resources:

- [JSON mappings](work-mappings.json)
- [sample events](work-events.json)

These mappings refer to work items. A work item is an item representing a literary work. The work's EID is provided in the item's `MetadataPart`.

## Item

As each work item has a metadata part, we map its `eid` metadatum as the work's ID. To make it a globally unique URI, the mapping builds it as follows:

1. prefix `itn:works/`
2. work metadata part's GUID
3. `/` + work EID

where (2) grants its uniqueness, and (1+3) its friendliness.

Also, this node is the subject of a triple telling that the work is a CIDOC-CRM E90 symbolic object.

>Note that here we pick the item's metadata part GUID rather than the item's GUID itself. Of course, nothing changes for the URI, as the GUID is opaque, and its only purpose is making the URI globally unique. Anyway, the part's GUID here is chosen to be consistent with event's related entities URIs, which refer to parts GUIDs, as nothing ensures that the related entity always corresponds to a whole item, rather then being defined in any of its parts only. Thus, always picking the part's GUID allows consistency, whatever the details of source data modeling, unless of course you are dealing with entities derived from items rather than from any of its parts.

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

Also, there is an additional child mapping for each type of relation in the list of the event's related entities. For instance, the _text sent_ event has these possible relations:

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

- a node for the recipient, whose URI is equal to the link's target GID.
- a triple telling that event P11 has participant recipient.

### Event Assertions

References for this section:

- [E13 Attribute Assignment | CIDOC CRM](https://cidoc-crm.org/Entity/E13-Attribute-Assignment/version-7.1).
- [E13 Attribute Assignment | CIDOC CRM issues](https://cidoc-crm.org/Issue/ID-367-e13-attribute-assignment)
- [RDF-star in ARCO and CIDOC CRM - lists.w3.org](https://lists.w3.org/Archives/Public/public-rdf-star-wg/2023May/0054.html).

Events may come with _assertions_, which define their level of probability with a numeric rank, optionally accompanied by documental references.

To represent this, first, we can create a custom `has_probability` property derived from `crm:P141_assigned`, and use it for a triple stating that the event has probability equal to some literal value:

- EVENT `itn:has_probability` "0.9"^^xsd:float.

>CIDOC-CRM allows users to define their own extensions or subproperties of existing properties to suit their needs. In this case, I used a subproperty of `P141_assigned` (was assigned by), which is a generic property for assigning any kind of attribute to an entity.

This property could be defined as:

```turtle
itn:has_probability a owl:DatatypeProperty ;
    rdfs:domain crm:E13_Attribute_Assignment ;
    rdfs:range xsd:float ;
    rdfs:subPropertyOf crm:P141_assigned ;
    rdfs:label "has probability" ;
    rdfs:comment "This property assigns a degree of probability to an event statement." .
```

Using RDF star, we could use this property like:

```turtle
<<:Leonardo_da_Vinci :painted :Mona_Lisa>> :has_probability "0.9"^^xsd:float .
<<:Leonardo_da_Vinci :painted :Mona_Lisa>> a crm:E13_Attribute_Assignment ;
    crm:P140_assigned_attribute_to :Mona_Lisa ;
    crm:P141_assigned :has_probability ;
    crm:P177_assigned_property_of_type crm:E55_Type .
```

>Here, we use an embedded triple to represent the event statement that Leonardo da Vinci painted the Mona Lisa, and assign a probability value of 0.9 to it using a custom property `:has_probability`. Then, we use `E13 attribute assignment` to describe the action of making this assertion, and use `P140`, `P141` and `P177` to link it to the involved entities and property type.

In our case, the event itself replaces the RDF star statement used as the subject.

Then, if we have further data about this assignment, which in our case is represented by _references_, we can create an _assertion_ entity, and say that:

- ASSERTION `a` `crm:E13_Attribute_Assignment`;
- ASSERTION `crm:P140_assigned_attribute_to` EVENT;
- ASSERTION `crm:P141_assigned` `itn:has_probability`;
- ASSERTION `crm:P177_assigned_property_of_type` `crm:E55_Type`.

>`E13_attribute_assignment` "comprises the actions of making assertions about one property of an object or any single relation between two items or concepts. The type of the property asserted to hold between two items or concepts can be described by the property `P177 assigned property type` : `E55 Type`. For example, the class describes the actions of people making propositions and statements during certain scientific/scholarly procedures, e.g., the person and date when a condition statement was made, an identifier was assigned, the museum object was measured, etc. [...] This class allows for the documentation of how the respective assignment came about, and whose opinion it was. Note that all instances of properties described in a knowledge base are the opinion of someone. Per default, they are the opinion of the team maintaining the knowledge base. This fact must not individually be registered for all instances of properties provided by the maintaining team, because it would result in an endless recursion of whose opinion was the description of an opinion. Therefore, the use of instances of E13 Attribute Assignment marks the fact, that the maintaining team is in general neutral to the validity of the respective assertion, but registers someone else’s opinion and how it came about".

Also, for each reference:

- REFERENCE `a E31_document`;
- REFERENCE `rdfs:label` "..."
- ASSERTION `P17_was_motivated_by` REFERENCE.

### Groups of Events

When events are grouped via a common `tag` in the context of the same events part, we want to represent them as parts of a bigger event. To this end, CIDOC-CRM provides `E4_Period`, which is a superclass of `E5_Event`.

A period may consist of (`P9_consists_of`) any number of other periods (or events, as events derive from periods); or, inversely, a period (or event) may form part of (`P9i_forms_part_of`) another period. For instance, a birth event might be decomposed into its various stages, eventually involving different actors.

So, whenever an event has a `tag` we will say that this event `P9i_forms_part_of` a new node of type `E4_Period`, whose URI will be built in such a way to be globally unique (e.g. with the part GUID and the tag value, given that event tags are assumed to be unique in the context of the same part).

### General Conventions

The entities involved in the events, and more generally in other parts, require a preliminary decision about editorial conventions. Most Cadmus models linking their data to entities outside the model itself do it via a link model which can be used both for external and internal links.

**External links** are just globally unique identifiers drawn from some LOD ontology (as URI, e.g. DBPedia) or any other external resources. For instance, <http://dbpedia.org/resource/Petrarch> could be the URI used for Petrarch. Anyway, the GID does not need to be a URI or any other globally unique identifier; it must be unique at least in the context of the database. In the end, an external link is just a user-defined string, with a corresponding label.

**Internal links** are links cross referencing two entities found in the Cadmus database, via entity IDs (EIDs). An EID can be assigned to an item as a whole (via its `MetadataPart`, by a convention assuming that the metadatum named eid refers to the item including that part), or to any entity defined in a part's model. The provided UI (from bricks) allows user to lookup these EIDs by just typing any of their characters, and eventually filter them by item and part.

In both cases, the [link model](https://github.com/vedph/cadmus-bricks-shell/blob/master/projects/myrmidon/cadmus-refs-asserted-ids/README.md#asserted-composite-id) is the same, and includes a GID, i.e. a globally unique ID, plus some metadata. For external links, the GID is just the ID entered by users. For internal links, the GID usually is generated by a convention which ensures both a human-readable and globally unique ID, by merging a GUID (from a part or item) with the human-friendly EID. For instance, a GID like:

```txt
P3e04b8d1-7b55-4de7-b2b8-3097017c7c82/petrarch
```

can be dissected as:

- a `P` prefix telling us that the GUID belongs to a part (for an item we would have `I`).
- the part's (or item's) GUID.
- the EID (here `petrarch`) prefixed by `/`.

>The UI also allows users to customize the GID, if your Cadmus project is configured for this (via a thesaurus).

As for **mapping**, the typical scenarios are:

(A) _copied external links_: true URIs or other global identifiers can be copied without changes. In this case, you are using a globally unique ID as your external link, like <http://dbpedia.org/resource/Petrarch>. So, the mapping will just assume this as the URI of the projected node.

(B) _mapped external links_: identifiers which are unique only within the editor database can be given a more graph-like ID, while still keeping them inside the project's namespace. For instance, say you want to use simple toponyms as place identifiers. Of course, you stick to some convention ensuring that all the place names you use are connected to a single place entity, disambiguating where necessary; and that all the place names are always spelled the same way. Apart from these obvious editorial conventions, you are then free to just type place names in the language and form which is considered standard for your project. For instance, we could just enter place names like `Roma`, `Arezzo`, etc. Then, a mapping rule might just add some prefix common to all place names, e.g. `itn:places/` for the _Itinera_ project, thus generating URIs like `itn:places/Roma`. Later, you might eventually alias these URIs by matching them with one or more gazetteer services, or a predefined list of some sort. This way, you will provide them with additional metadata like geographical location etc., while still keeping data entry as friendly as possible.

(C) _mapped internal links_: internal links will be mapped in some way to generate URIs from them. For instance, say you used a link to refer to Petrarch. In this case, the link model will include not only a preset internal GID as explained above; but also all the metadata referred to the pin-based lookup operation which led to target Petrarch: item ID, part ID, pin name, pin value, etc. So, your mapping can use such metadata to build a URI like e.g. `itn:persons/3e04b8d1-7b55-4de7-b2b8-3097017c7c82/petrarch`, which is built from:

1. a prefix common to all the persons in our database (`itn:persons/`);
2. the GUID of the item corresponding to the Petrarch entity; this GUID ensures that the resulting URI is globally unique;
3. the EID of the item corresponding to the Petrarch entity (`petrarch`), which makes the URI human-friendly.

So, instead of having a mapping like this, which just copies the GID as for case (A) above (`"recipient": "{@id.target.gid}"`):

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

we would have:

```json
{
  "name": "text sent event/related/has_participant",
  "source": "relatedEntities[?relation=='text:send:recipient']",
  "output": {
    "nodes": {
      "recipient": "itn:persons/{@id.target.itemId}/{@id.target.value}"
    },
    "triples": ["{?event} crm:P11_has_participant {?recipient}"]
  }
}
```

where the recipient URI is built exactly as explained in the above sample: a prefix, the item's GUID, and the item's EID (which corresponds to the name of the pin, as the item's EID comes from the `eid` metadatum of the metadata part for that item): `"recipient": "itn:persons/{@id.target.itemId}/{@id.target.value}"`.

Of course, this means that you might have different mappings when mixing both external and internal identifiers in the same category of links. As an internal link can be easily distinguished from an external one by the presence of pin-only properties like the pin's `name`, you can have two mappings for the same related entity, one with an internal link and another with an external link; and generate the corresponding URI in different ways.

As a sample, consider this event mapping with a single related entity type (`text:send:recipient`) for brevity:

```json
{
  "name": "text sent event",
  "sourceType": 2,
  "facetFilter": "work",
  "partTypeFilter": "it.vedph.historical-events",
  "description": "Map text sent event",
  "source": "events[?type=='text.send']",
  "sid": "{$part-id}/{@[0].eid}",
  "output": {
    "metadata": {
      "id": "{$part-id}/{@eid}",
      "work": "itn:works/{$metadata-pid}/{$item-eid}"
    },
    "nodes": {
      "event": "itn:events/{$id} itn:events/{@eid}"
    },
    "triples": [
      "{?event} a crm:E7_Activity",
      "{?event} crm:P2_has_type itn:event-types/text.send",
      "{?event} crm:P16_used_specific_object {$work}"
    ]
  },
  "children": [
    {
      "name": "event_description"
    },
    {
      "name": "event_note"
    },
    {
      "name": "event_chronotopes"
    },
    {
      "name": "text sent event/related/has_participant",
      "source": "relatedEntities[?relation=='text:send:recipient' && !id.target.name]",
      "output": {
        "nodes": {
          "recipient": "{@id.target.gid}"
        },
        "triples": ["{?event} crm:P11_has_participant {?recipient}"]
      }
    },
    {
      "name": "text sent event/related/has_participant",
      "source": "relatedEntities[?relation=='text:send:recipient' && id.target.name]",
      "output": {
        "nodes": {
          "recipient": "itn:persons/{@id.target.partId}/{@id.target.value}"
        },
        "triples": ["{?event} crm:P11_has_participant {?recipient}"]
      }
    }
  ]
}
```

In these mappings, you should notice that:

- the work URI is built from prefix `itn:works/` + the GUID of the metadata part for the work item + `/` + item's EID.
- there are 2 children mappings for the recipient entity: one matches the recipient ID represented by an external link (`!id.target.name` = there is no target name in the ID), another matches the recipient ID represented by an internal link (`id.target.name` = there is a target name in the ID, which means that this is a pin-based link, i.e. internal).

>As noted above, the target URI is built with the _part_ ID rather than with the _item_ ID. We could also use the `itemId`, but only if we are sure that all the related entities correspond to items. If instead an entity is defined only in a part, it would make no sense to target its container item; sure, the resulting URI would be globally unique, but in practice we would lose the direct connection between the link and its target part. So, the general rule is just sticking to the most granular GUID, i.e. use part ID unless not present.

### Itinera Conventions

In _Itinera_, there are 3 types of links:

- **internal**: an internal link between two entities in the database. This is marked by the presence of `id.target.name`.
- **external**: marked by the absence of `id.target.name`, divided in:
  - **global** link: a true external link, like a DBPedia URI. In Itinera this is a marked case, and is distinguished by local links by a `@` prefix before the ID.
  - **local** link: a link which does not refer to a database entity, but is local to the database context, e.g. `scipione_barbato`. This is a conventional identifier which makes sense and is unique only within the context of the Itinera project, though not referred to an entity existing in the database.

So, the match conditions are:

- internal: `relatedEntities[?relation=='THESAURUS_ID' && id.target.name`
- external, global: `relatedEntities[?relation=='THESAURUS_ID' && !(id.target.name) && starts_with(id.target.gid, '@')]`. In this case, we will have to remove the `@` prefix when mapping, e.g. with `slice(target.gid, 1)`.
- external, local: `relatedEntities[?relation=='THESAURUS_ID' && !(id.target.name) && !(starts_with(id.target.gid, '@'))]`
