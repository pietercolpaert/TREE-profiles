# TREE-profiles

Profiles for serializations such as n-triples, turtle and TRiG to first contain a group for the hypermedia, and then contain groups for the members.

The profile allows TREE clients to speed up parsing of individual members.

## New profiles for the content-type header

Servers add `;profile="https://w3id.org/tree/profile"` to the content-type headers on top of their Turtle (or the likes) responses, such as `Content-Type: text/turtle;profile="https://w3id.org/tree/profile`.

For the highest performance, pick `Content-Type: application/n-quads;profile="https://w3id.org/tree/profile`.

In the TREE profile we propose to speed up the parsing by indicating to a parser that:
 * All triples related to the hypermedia MUST be grouped together at the beginning of the document.
 * Every subsequent group MUST contain all quads of a specific member, starting with the triple linking the member to the collection.

## An example

```turtle
@prefix ex: <https://example.org/> .
@prefix tree: <https://w3id.org/tree/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>.

# The document starts with hypermedia controls
<> tree:relation [
    a tree:GreaterThanOrEqualToRelation ;
    tree:node <?greaterThanOrEqualTo=10>;
    tree:value 10;
    tree:remainingItems 10 ;
    tree:path ex:value
] .

ex:Collection1 a tree:Collection;
            tree:view <> ;
            rdfs:label "A Collection of 2 subjects"@en;
# `tree:member` indicates the hypermedia group is done, and the first member begins
            tree:member ex:Subject1 .

ex:Subject1 a ex:Subject ;
            rdfs:label "Subject 1" ;
			ex:value 2 ;
            ex:linkedTo [ a ex:Subject ] .

# Start of the second member
ex:Collection1 tree:member ex:Subject2 .
ex:Subject2 a ex:Subject ;
            rdfs:label "Subject 2" ;
			ex:value 9 ;
            ex:linkedTo ex:Subject1 .
# EOF flags the end of the second member
```

## Implementing a TREE profile client

When the `content-type` response header indicates the TREE profile, the client MAY ignore the member extraction algorithm in the TREE specification.
Every time a `tree:member` triple is encountered, the subsequent quads until the next `tree:member` triple or the end of the file (EOF) are part of that member.
