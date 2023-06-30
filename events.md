# Historical Events Mappings

Events related to the work are listed in the work item's _events part_. For each **event**, the event _type_ is provided by a thesaurus entry, and so are the relation types for that event with reference to directly related entities.

In general:

- each event corresponds to an entity node, whose URI is `itn:events/PART-GUID/EID`.
- each event has at least 3 triples where the event is the _subject_; these triples say that:
  - event is an _E7 activity_.
  - event _P2 has type_ + the event's type as derived from the thesaurus entry. The entity for this entry has URI `itn:event-types/ID`.
  - event _P16 used specific object_ + the work entity. This connects the event with its work.

>The work object for the last triple has a URI corresponding to the URI built for the work item: this is why we are using the supplied metadatum `metadata-pid`, which is the ID (GUID) of the metadata part contained by the work's item. So, `"work": "itn:works/{$metadata-pid}/{$item-eid}"` in this mapping builds the same URI as `"work": "itn:works/{$part-id}/{@value} itn:works/{@value}"` in the work mapping. In fact, the work mapping just matches the work item's metadata part, so in this context the more usual `part-id` metadatum is equivalent to `metadata-pid`, which is used when the part being matched is different (here it's the events part).

Also, a number of **children mappings** are provided for each event mapping. The following mappings are always the same, and thus shared among all the event mappings:

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

## Event Assertions

References for this section:

- [E13 Attribute Assignment | CIDOC CRM](https://cidoc-crm.org/Entity/E13-Attribute-Assignment/version-7.1).
- [E13 Attribute Assignment | CIDOC CRM issues](https://cidoc-crm.org/Issue/ID-367-e13-attribute-assignment)
- [RDF-star in ARCO and CIDOC CRM - lists.w3.org](https://lists.w3.org/Archives/Public/public-rdf-star-wg/2023May/0054.html).

Events may come with _assertions_, which define their level of probability with a numeric rank, optionally accompanied by documental references. The model for assertions in Cadmus is as follows:

- `rank` (`short`): a numeric rank, usually ranging from 0 to N, where 0=unspecified and any other positive integer number is a rank value (1=most probable, 2=less probable than 1, etc.).
- `tag` (`string`): an optional tag for grouping or classifying the assertion in some way.
- `note` (`string`): an optional short free note.
- `references` (`DocReference[]`): document references for the assertion:
  - `type` (`string`): optional document type (e.g. bibliography, archive document, manuscript, etc.).
  - `tag` (`string`): an optional tag for grouping or classifying the reference in some way.
  - `citation`\* (`string`): the reference value. Its structure often changes according to its `type`. For instance, a bibliography reference might be as simple as `Rossi 1963` (e.g. surname and publication year).

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

So we can create an _assertion_ entity, and say that:

- ASSERTION `a` `crm:E13_Attribute_Assignment`;
- ASSERTION `crm:P140_assigned_attribute_to` EVENT;
- ASSERTION `crm:P141_assigned` `itn:has_probability`;
- ASSERTION `crm:P177_assigned_property_of_type` `crm:E55_Type`.

>`E13_attribute_assignment` "comprises the actions of making assertions about one property of an object or any single relation between two items or concepts. The type of the property asserted to hold between two items or concepts can be described by the property `P177 assigned property type` : `E55 Type`. For example, the class describes the actions of people making propositions and statements during certain scientific/scholarly procedures, e.g., the person and date when a condition statement was made, an identifier was assigned, the museum object was measured, etc. [...] This class allows for the documentation of how the respective assignment came about, and whose opinion it was. Note that all instances of properties described in a knowledge base are the opinion of someone. Per default, they are the opinion of the team maintaining the knowledge base. This fact must not individually be registered for all instances of properties provided by the maintaining team, because it would result in an endless recursion of whose opinion was the description of an opinion. Therefore, the use of instances of E13 Attribute Assignment marks the fact, that the maintaining team is in general neutral to the validity of the respective assertion, but registers someone else’s opinion and how it came about".

Also, for each reference citation:

- CITATION `a E31_document`;
- CITATION `rdfs:label` "...citation value...";
- ASSERTION `P70i_is_documented_in` CITATION.

## Groups of Events

When events are grouped via a common `tag` in the context of the same events part, we want to represent them as parts of a bigger event. To this end, CIDOC-CRM provides `E4_Period`, which is a superclass of `E5_Event`.

A period may consist of (`P9_consists_of`) any number of other periods (or events, as events derive from periods); or, inversely, a period (or event) may form part of (`P9i_forms_part_of`) another period. For instance, a birth event might be decomposed into its various stages, eventually involving different actors.

So, whenever an event has a `tag` we will say that this event `P9i_forms_part_of` a new node of type `E4_Period`, whose URI will be built in such a way to be globally unique, here using the constant prefix `itn:periods`, part GUID and tag value, all separated by slash (event tags are assumed to be unique in the context of the same part).
