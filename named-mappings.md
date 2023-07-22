# Named Mappings

Named mappings are just a shortcut used in JSON documents representing mappings to avoid repeating the same children mappings in many different places. So, they are a way of making these documents more compact and readable.

Currently, all these mappings refer to events, and _assume the following output_ provided by their ascendants:

- a _metadatum_ `sid` as the event's SID (this will be used for more suffixed SIDs);
- a _node_ named `event` as the subject.

## Assertion

`event_assertion` matches `assertion`. It emits an assertion node with a number of triples for it and its optional references.

A variant of this mapping used for works is `work_assertion`, which is equal to `event_assertion` except that it uses a metadatum named `event` for the event (rather than a node).

## Chronotopes

`event_chronotopes` matches `chronotopes`. For each chronotope, it matches `place` and `date`; both these create a node (for place and timespan) and some triples for them.

## Description and Note

`event_description` and `event_note` match `description` and `note` respectively, and map them into triple `EVENT P3_has_note LITERAL`.

## Period

`event_period` matches `tag`. It emits a period node with triple `EVENT P9i_forms_part_of PERIOD`.
