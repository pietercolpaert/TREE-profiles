# TREE-N3-profiles

Profiles for N3 serializations such as turtle and TRiG to first contain a group for the hypermedia, and then contain groups for the members.

The profile allows TREE clients to speed up parsing of individual members.

## New Mime Types

 * `application/tree+trig`
 * `text/tree+turtle`
 * `application/tree+n-triples`
 * `application/tree+n-quads`

## Using parser directives

The `tree+` indicates to the client the fact the server grouped quads together using parser directives or pragmas.
The group pragma looks as follows:

```turtle
# @group begin
<> <p> <o> .
# @group end
```

All triples or quads in-between this pragma will be emitted by a group parser (such as [Wurtle.js](https://github.com/pietercolpaert/Wurtle.js)).

## The TREE profile

TREE is a hypermedia specification for handling collections of entities. Each HTTP response will contain hypermedia controls and members of the collection.
In the TREE profile we propose to speed up the parsing by indicating to a parser that:
 * All triples related to the hypermedia MUST be grouped together in the first group that gets parsed.
 * Every subsequent group MUST contain all quads of a specific member, plus the triple linking the member to the collection.

## An example

```turtle

# @group begin
ex:Collection1 a tree:Collection;
            rdfs:label "A Collection of 2 subjects"@en;
# @group end
# @group begin
            tree:member ex:Subject1;
ex:Subject1 a ex:Subject ;
            rdfs:label "Subject 1" ;
            ex:linkedTo [ a ex:Subject ] .
# @group end
# @group begin
ex:Collection1 tree:member ex:Subject2 .
ex:Subject2 a ex:Subject ;
            rdfs:label "Subject 2" ;
            ex:linkedTo ex:Subject3 .
# @group end
```

## Implementing a TREE client

A TREE client SHOULD add `application/tree+trig` to their `Accept` request header.
When the `content-type` response header indicates one of the TREE profile mime-types, the client SHOULD use a grouped quads parser.
When it does, then the client will first receive a group that contains the hypermedia.
All subsequent group will contain the members.

For group1, the TREE specification on search forms and traversal can be followed using these triples only.

For subsequent groups, we will adapt the member extraction algorithm.
For each triple in the array, the client MUST check whether `<yourcollection> tree:member ?m` has been set.
If it is, the client MUST remove that triple for the array and emit ?m as the member id, and the array as quads of the member.
If it is not, the client MUST ignore the current group as it MAY be used for something else beyond this specification.
