# Introduction

**JSKOS** defines a JavaScript Object Notation (JSON) structure to encode
knowledge organization systems, such as classifications, thesauri, and
authority files. The current draft of JSKOS supports encoding of [concepts] and
[concept schemes] with their corresponding properties. Support of [concept
mappings] and [concept collections] is experimental.

The main part of JSKOS is compatible with Simple Knowledge Organisation System
(SKOS) and JavaScript Object Notation for Linked Data (JSON-LD) but JSKOS can
be used without having to be experienced in any of these technologies. A simple
JSKOS document can be mapped to SKOS expressed in the Resource Description
Framework (RDF), and vice versa. An extended JSKOS document may further include
[closed world statements] without correspondence in RDF. This feature
especially allows for use of knowledge organization systems in web
applications.

JSKOS is currently being developed as part of [ng-skos] but it can be used
independently. The specification is hosted at <http://gbv.github.io/jskos/> in
the public GitHub repository <https://github.com/gbv/jskos>. Feedback is
appreciated!

# Concepts
[concept]: #concepts
[concepts]: #concepts

A **concept** represents a [SKOS Concept](http://www.w3.org/TR/skos-primer/#secconcept).
A concept is a JSON object with the following optional properties:

name         |type                       |description
-------------|---------------------------|-------------------------------------------------------------------------------
uri          |string                     |URI of the concept
type         |array of string            |URIs of RDF types (first must be `http://www.w3.org/2004/02/skos/core#Concept`)
notation     |array of strings           |list of notations
prefLabel    |object of strings          |preferred concept labels, index by language
altLabel     |object of arrays of strings|alternative concept labels, indexed by language
hiddenLabel  |object of arrays of strings|hidden concept labels, indexed by language
narrower     |boolean or array of objects|narrower concepts
broader      |boolean or array of objects|broader concepts
related      |boolean or array of objects|related concepts
ancestors    |boolean or array of objects|list of ancestors, possibly up to a top concept
inScheme     |array of strings or objects|[concept schemes] or URI of the concept schemes
topConceptOf |array of strings or objects|[concept schemes] or URI of the concept schemes
scopeNote    |object of arrays of strings|see [SKOS Documentary Notes]
definition   |object of arrays of strings|see [SKOS Documentary Notes]
example      |object of arrays of strings|see [SKOS Documentary Notes]
historyNote  |object of arrays of strings|see [SKOS Documentary Notes]
editorialNote|object of arrays of strings|see [SKOS Documentary Notes]
changeNote   |object of arrays of strings|see [SKOS Documentary Notes]
@context     |string                     |URI referencing a JSKOS [JSKOS-LD context] document

All properties are optional, so the empty object `{}` is also a valid concept.
Additional properties, not included in this list, should be ignored.

Applications MAY use only the first element of property `notation` and/or property
`inScheme` for simplification.

<div class="note">
The "ancestors" field only makes sense monohierarchical classifications but
it's not forbidden to choose just one arbitrary path of concepts that are
connected by the narrower relation.
</div>

<div class="example">
```json
{
  "uri": "http://example.org/terminology/P",
  "type": ["http://www.w3.org/2004/02/skos/core#Concept"],
  "scopeNote": {
    "en": "state of harmony characterized by lack of violent conflict and freedom from fear of violence",
    "de": "Abwesenheit von Gewalt, Angst und anderen Störungen"
  },
  "prefLabel": {
    "en": "peace",
    "de": "Frieden"
  },
  "altLabel": {
    "de": ["Friede"]
  },
  "notation": ["P"],
  "narrower": [
    {
      "prefLabel": {
        "en": "world peace",
        "de": "Weltfrieden"
      } 
    }
  ],
  "related": [
    { 
      "prefLabel": { "en": "war", "de": "Krieg" } 
    }
  ]
}
```
</div>

The order of alternative labels with same language and the order of narrower,
broader, or related concepts is irrelevant, but this may be changed (see
<https://github.com/gbv/jskos/issues/11>).

# Concept Schemes
[concept scheme]: #concept-schemes
[concept schemes]: #concept-schemes

A **concept scheme** represents a [SKOS Concept Scheme].
A concept scheme is a JSON object with the following optional properties:

property   |type                       |definition
-----------|---------------------------|-------------------------------------------------------------------------------------
uri        |string                     |URI of the concept scheme
type       |array of string            |URIs of RDF types (first must be `http://www.w3.org/2004/02/skos/core#ConceptScheme`)
notation   |array of strings           |list of acronyms or notations of the concept scheme
prefLabel  |object of strings          |preferred titles of the concept scheme, index by language
altLabel   |object of arrays of strings|alternative titles of the concept scheme, indexed by language
hiddenLabel|object of arrays of strings|hidden titles of the concept scheme, indexed by language
topConcepts|array of objects           |top concepts of the concept scheme
@context   |string                     |URI referencing a JSKOS [JSKOS-LD context] document

All properties are optional, so the empty object `{}` is also a valid concept
scheme. Additional properties, not included in this list, should be ignored.

<div section="note">
notation and label properties do not imply a domain, so they can be used for both, concepts and concept schemes.
</div>


# Mappings
[mappings]: #mappings
[concept mappings]: #mappings

A **mapping** represents a mapping between [concepts] of two [concept schemes].

<div class="note">
This section is highly experimental. Support of encoding mappings in JSKOS, based on 
[SKOS mapping properties](http://www.w3.org/TR/skos-reference/#mapping)
is planned. See <https://github.com/gbv/jskos/issues/8> for discussion.
</div>

A mapping is a JSON object with the following properties:

* mandatory unique identifier as URI (`uri`). This may also be a non-HTTP URI,
  such as `urn:uuid:687b973c-38ab-48fb-b4ea-2b77abf557b7`
* optional DCMI Metadata Terms for metadata about the mapping, such as 
  `creator`, `contributor`, `publisher`, `dateAccepted`, `modified`, 
  `source`, `provenance`...
* property `mappingType` indicating one of the 
  [SKOS mapping properties](http://www.w3.org/TR/skos-reference/#mapping)
  `closeMatch`, `exactMatch`, `broadMatch`, `narrowMatch`, `relatedMatch`.
* property `mappingRelevance` with a numerical value between 0 and 1.
* mandatory properties `from` and `to` with [concept bundles] of
  source and target scheme. 

A mapping should either include property `mappingType` or property
`mappingRelevance`. If neither of both is given, the mappingType `closeMatch`
can be assumed as default.


# Concept Bundles
[concept bundles]: #concept-bundles
[collections]: #concept-bundles
[concept collections]: #concept-bundles

A **concept bundle** is a group of [concepts]. Some concept bundles represent
[SKOS concept collections](http://www.w3.org/TR/skos-reference/#collections) but
bundles may serve other purposes as well.

<div class="note">
Concept bundles are highly experimental. 
See <https://github.com/gbv/jskos/issues/7> for discussion.
</div>

A concept bundle is a JSON object with the following properties. All properties
are optional except one of `conceptSet` or `conceptList` but not both must be
given. 

property     | type                        | definition
-------------|-----------------------------|-----------------------------------------------------
conceptSet   | array or strings or objects | set of [concepts] or URIs of concepts
conceptList  | array or strings or objects | list of [concepts] or URIs of concepts
inScheme     | array or strings or objects | set of [concept schemes] or URIs of concept schemes
coordination | string                      | the value `AND` or the value `OR`

<div class="note">

* More possible coordination values may be added for support of more advanced
structured indexing (ways to relate concepts to one another).

* Concepts from a bundle may also come from different concept schemes!

* A concept bundle may be empty, for instance to indicate that no appropriate
  concepts exists for a given concept scheme:

    ```json
    { 
      "conceptSet": [ ],
      "inScheme": [ "http://dewey.info/scheme/ddc/" ]
    }
    ```
</div>


# Closed world statements
[closed world statements]: #closed-world-statements

By default, an JSKOS document should be interpreted as possible incomplete: a
missing property does not imply that no value exists for this property: this
assumption is also known as open-world assumption. A JSKOS document may include
special closed world statements to explicitly disable the open world assumption
for selected properties.

Applications may use empty arrays, objects, or boolean values (`true` and
`false`) to explicitly state the known absence or existence of the following
properties:

property  | explicit negation | explicit existence
----------|-------------------|-------------------
notation  | `[ ]` or `false`  | `true`
prefLabel | `{ }` or `false`  | `true`
altLabel  | `{ }` or `false`  | `true`
narrower  | `[ ]` or `false`  | `true`
broader   | `[ ]` or `false`  | `true`
ancestors | `[ ]` or `false`  | `true`
related   | `[ ]` or `false`  | `true`

<div class="example">
The following concept has at least one preferred label but no alternative
labels, notations, nor narrower concepts. Nothing is known about broader
concepts, related concepts, and other possible concept properties:

```json
{
  "type": ["http://www.w3.org/2004/02/skos/core#Concept"],
  "prefLabel": true,
  "altLabel": { },
  "notation": [ ],
  "narrower": false
}
```
</div>

<div class="note">
By now it is *not possible* to indicate the existence of an unknown URI, 
unknown preferred labels, and an unknown concept scheme. The following
key-value  pairs are **not allowed**, each:

```json
{
  "uri": true,
  "prefLabel": { "en": false, "es": true },
  "altLabel": { "en": [] },
  "inScheme": true
}
```

Support for closed-world-statements on missing languages
(see <https://github.com/gbv/jskos/issues/5>) and on the completeness of sets
(see <https://github.com/gbv/jskos/issues/6>) may be added in a later version 
of this specification.
</div>

<div class="note">
Closed world statements can be removed from a JSKOS document by removing 
all boolean values:

```javascript
function removeClosedWorldStatements(jskos) {
    if (Array.isArray(jskos)) {
        var i = 0;
        while (i < jskos.length) {
            if (jskos[i] === true || jskos[i] === false) {
                jskos.splice(i, 1);
            } else {
                removeClosedWorldStatements(jskos[i]);
                i++;
            }
        }
    } else if (typeof jskos == "object") {
        for (var p in jskos) {
            if (jskos[p] === true || jskos[p] === false) {
                delete jskos[p];
            } else {
                removeClosedWorldStatements(jskos[p]);
            }
        }
    }
}
```
</div>

# Integrity rules

Integrity rules of SKOS should be respected. A later version of this specification
may list these rules in more detail.

[RDF/SKOS]: http://www.w3.org/2004/02/skos/
[ng-skos]: http://gbv.github.io/ng-skos/
[SKOS Concept Scheme]: http://www.w3.org/TR/skos-primer/#secscheme 
[JSKOS-LD context]: #json-ld-context
[SKOS Documentary Notes]: http://www.w3.org/TR/skos-primer/#secdocumentation

# References {.unnumbered}

## Normative references {.unnumbered}

* A. Phillips, M. Davis: *Tags for Identifying Languages.*
  IETF Best Current Practice BCP 47, September 2009
  <http://tools.ietf.org/html/bcp47>

* S. Bradner: *Key words for use in RFCs to Indicate Requirement Levels*. 
  RFC 2119, March 1997. <https://tools.ietf.org/html/rfc2119>

* M. Dürst, M. Suignard: *Internationalized Resource Identifiers (IRIs)*. 
  RFC 3987, January 2005. <https://tools.ietf.org/html/rfc3987>

* D. Crockford: *The application/json Media Type for JavaScript Object Notation (JSON)*.
  RFC 4627, July 2006. <https://tools.ietf.org/html/rfc4627>

## Informative references {.unnumbered}

* Alistair Miles, Sean Bechhofer (Editors): *SKOS Reference*. W3C
  Recommendation, 18 August 2009. <http://www.w3.org/TR/skos-reference>

* Jakob Voß, Moritz Horn: *ng-skos*. AngularJS module.  
  <https://github.com/gbv/ng-skos>.

* Manu Sporny, Dave Longley, Gregg Kellogg, Markus Lanthaler, 
  Niklas Lindström (Editors): *JSON-LD 1.0*. W3C
  Recommendation, 16 January 2014. <http://www.w3.org/TR/json-ld/>

# Appendices {.unnumbered}

The following appendices are *non-normative*.

## SKOS features not supported in JSKOS {.unnumbered}

JSKOS is aligned with SKOS but all references to SKOS are informative only.
The following features of SKOS are not supported in JSKOS (yet):

will be supported
  : * [mapping properties], see <https://github.com/gbv/jskos/issues/8>
maybe supported later
  : * [concept collections], see <https://github.com/gbv/jskos/issues/7>
    * [closed world statements] about missing or more languages,
      see <https://github.com/gbv/jskos/issues/5>
    * [closed world statements] about missing or complete sets,
      see <https://github.com/gbv/jskos/issues/6>
will not be supported
  : * datatypes of notations (of little use in practice)
    * labels and notes without language tag (rarely used in practice)
    * skos:semanticRelation (can be derived)
    * skos:narrowerTransitive (can be derived)

[documentation properties]: http://www.w3.org/TR/2009/REC-skos-reference-20090818/#notes
[mapping properties]: http://www.w3.org/TR/2009/REC-skos-reference-20090818/#mapping

## JSKOS features not supported in SKOS {.unnumbered}

The following features of JSKOS have no corresponce in SKOS:

* [closed world statements](#closed-world-statements)
* order of broaderTransitive statements (can be derived)
* order of multiple notations
* order of multiple inScheme statements

## JSON-LD context {.unnumbered}

The following [JSON-LD context document] can be used to map JSKOS to map JSKOS
without [closed world statements] to RDF triples. 

[JSON-LD context document]: http://www.w3.org/TR/json-ld/#the-context

```json
{
    "skos": "http://www.w3.org/2004/02/skos/core#",
    "uri": "@id",
    "type": "http://www.w3c.org/1999/02/22-rdf-syntax-ns#type",
    "notation": "skos:notation",
    "prefLabel": {
        "@id": "skos:prefLabel",
        "@container": "@language"
    },
    "altLabel": {
        "@id": "skos:altLabel",
        "@container": "@language"
    },
    "hiddenLabel": {
        "@id": "skos:altLabel",
        "@container": "@language"
    },
    "narrower": "skos:narrower",
    "broader": "skos:broader",
    "related": "skos:related",
    "ancestors": "skos:broaderTransitive",
    "inScheme": {
        "@id": "skos:inScheme",
        "@type": "@id"
    },
    "topConceptOf": {
        "@id": "skos:topConceptOf",
        "@type": "@id"
    },
    "topConcepts": "skos:hasTopConcept"
}
```

JSKOS with closed world statements can be mapped to RDF by ignoring all boolean
values and/or by mapping selected boolean values to RDF triples with blank
nodes.

Applications should further add implicit RDF triples, such as `$someConcept
rdf:type skos:Concept`, if such information can be derived from JSKOS by other
means.

# Examples  {.unnumbered}

## Integrated Authority File (GND) {.unnumbered}

The Integrated Authority File (German: *Gemeinsame Normdatei*) is an authority
file managed by the German National Library.

<div class="example">
GND as as JSKOS concept scheme:

```json
{
    "uri": "http://d-nb.info/gnd/7749153-1", 
    "type": ["http://www.w3.org/2004/02/skos/core#ConceptScheme"],
    "notation": ["GND"],
    "prefLabel": {
        "de": "Gemeinsame Normdatei",
        "en": "Integrated Authority File",
        "ar": "ملف استنادي متكامل",
        "ko": "게마인자메 노름다타이"
    }
}
```
</div>

<div class="example">
A concept from GND:

```json
{
    "uri": "http://d-nb.info/gnd/4074195-3",
    "type": ["http://www.w3.org/2004/02/skos/core#Concept"],
    "prefLabel": {
        "de": "Leukozyt"
    },
    "altlabel": {
        "de": [ "Leukocyt", "Weißes Blutkörperchen", "Leukozyten", "Leukocyten" ]
    },
    "broader": [ {
        "uri": "http://d-nb.info/gnd/4130604-1",
        "prefLabel": { "de": "Blutzelle" }
      } ],
    "narrower": [ {
        "uri": "http://d-nb.info/gnd/4036762-9",
        "prefLabel": { "de": "Lymphozyt" }
      }, {
        "uri": "http://d-nb.info/gnd/4158047-3",
        "prefLabel": { "de": "Granulozyt" }
      }, {
        "uri": "http://d-nb.info/gnd/4285013-7",
        "prefLabel": { "de": "Monozyt" }
     } ]
}
```
</div>

## Dewey Decimal Classification (DDC)  {.unnumbered}

<div class="example">
A concept from the Dewey Decimal Classification, German edition 22:

```json
{
    "uri": "http://dewey.info/class/612.112/e22/2014-04-15/", 
    "type": ["http://www.w3.org/2004/02/skos/core#Concept"],
    "notation": ["612.112"],
    "prefLabel": { "de": "Leukozyten (Weiße Blutkörperchen)" },
    "broader": [ {
        "notation": ["612.11"],
        "prefLabel": { "de": "Blut" },
        "uri": "http://dewey.info/class/612.11/e22/2014-04-15/"
      } ],
    "narrower": [ {
        "notation": ["612.1121"], 
        "prefLabel": { "de": "Biochemie" },
        "uri": "http://dewey.info/class/612.1121/e22/2014-04-15/" 
      },{
        "notation": ["612.1127"],
        "prefLabel": { "de": "Anzahl und Auszählung" },
        "uri": "http://dewey.info/class/612.1127/e22/2014-04-15/" 
      } ],
    "ancestors": [ { 
        "notation": ["612.11"], 
        "prefLabel": { "de": "Blut" }, 
        "uri": "http://dewey.info/class/612.11/e22/2014-04-15/"
      },{
        "notation": ["612.1"],
        "prefLabel": { "de": "Blut und Kreislauf" },
        "uri": "http://dewey.info/class/612.1/e22/2014-04-15/"
      },{
        "notation": ["612"],
        "prefLabel": { "de": "Humanphysiologie" },
        "uri": "http://dewey.info/class/612/e22/2014-04-15/"
      },{
        "notation": ["61"],
        "prefLabel": { "de": "Medizin & Gesundheit" },
        "uri": "http://dewey.info/class/61/e22/2014-04-15/"
      },{
        "notation": ["6"],
        "prefLabel": { "de": "Technik, Medizin, angewandte Wissenschaften" },
        "uri": "http://dewey.info/class/6/e22/2014-04-15/"
      } ],
    "inScheme": [
        "http://dewey.info/scheme/version/e22/2014-04-15/",
        "http://dewey.info/scheme/edition/e22/",
        "http://dewey.info/scheme/ddc/"
    ]
}
```
</div>

<div class="example">
A concept from the abbbridget Dewey Decimal Classification, edition 23, in three languages:

```json
{
    "uri": "http://dewey.info/class/641.5/e23/",
    "type": ["http://www.w3.org/2004/02/skos/core#Concept"],
    "notation": ["641.5"],
    "inScheme": ["http://dewey.info/edition/e23/"],
    "prefLabel": {
        "en": "Cooking",
        "de": "Kochen",
        "it": "Cucina"
    },
    "broader": [
        {
            "uri": "http://dewey.info/class/641/e23/",
            "notation": ["641"],
            "prefLabel": {
                "en": "Food and drink",
                "de": "Essen und Trinken",
                "it": "Cibi e bevande"
            }
        }
    ],
    "narrower": [
        {
            "uri": "http://dewey.info/class/641.502/e23/",
            "notation": ["641.502"],
            "prefLabel": {
                "en": "Miscellany",
                "de": "Verschiedenes",
                "it": "Miscellanea"
            }
        },
        {
            "uri": "http://dewey.info/class/641.508/e23/",
            "notation": ["641.508"],
            "prefLabel": {
                "en": "Cooking with respect to kind of persons",
                "de": "Kochen im Hinblick auf Personengruppen",
                "it": "Cucina in riferimento a categorie di persone"
            }
        },
        {
            "uri": "http://dewey.info/class/641.509/e23/",
            "notation": ["641.509"],
            "prefLabel": {
                "en": "Historical, geographic, persons treatment",
                "de": "Hostorische, geographische, personenbezogene Behandlung",
                "it": "Storia, geografia, persone"
            },
            "narrower": [
                {
                    "uri": "http://dewey.info/class/641.5092/e23/",
                    "notation": ["641.5092"],
                    "prefLabel": {
                        "en": "Cooks",
                        "de": "Köche",
                        "it": "Cuochi"
                    }
                }
            ]
        },
        {
            "uri": "http://dewey.info/class/641.59/e23/",
            "notation": ["641.59"],
            "prefLabel": {
                "en": "Cooking characteristic of specific geographic environments, ethnic cooking",
                "de": "Merkmale der Küche einzelner geografischer Umgebungen, ethnische Küche",
                "it": "Cucina tipica di specifici ambienti geografici, cucina etnica"
            }
        }
    ]
}
```
</div>

## Mappings {.unnumbered}

<div class="example">
Multiple mappings from one concept (612.112 in DDC) to GND.

```json
{
    "from": {
        "inScheme": ["http://dewey.info/scheme/edition/e22/"],    
        "conceptSet": [ {
            "uri": "http://dewey.info/class/612.112/e22/2014-04-15/",
            "notation": ["612.112"]
        } ]
    },
    "to": {
        "inScheme": [],
        "conceptSet": [ {
            "uri": "http://d-nb.info/gnd/4074195-3",
            "preflabel": { "de": "Leukozyt" }
        } ]
    },
    "mappingType": "closeMatch"
}
```

```json
{
    "from": {
        "inScheme": ["http://dewey.info/scheme/edition/e22/"],    
        "conceptSet": [ {
            "uri": "http://dewey.info/class/612.112/e22/2014-04-15/",
            "notation": ["612.112"]
        } ]
    },
    "to": {
        "inScheme": ["http://d-nb.info/gnd/7749153-1"],
        "conceptSet": [ {
            "uri": "http://d-nb.info/gnd/4074195-3",
            "preflabel": { "de": "Leukozyt" }
          },{
            "uri": "http://d-nb.info/gnd/4141893-1",
            "preflabel": { "de": "Alkalische Leukozytenphosphatase" }
          },{
            "uri": "http://d-nb.info/gnd/7606617-4",
            "preflabel": { "de": "Blutlymphozyt" }
          },{
            "uri": "http://d-nb.info/gnd/4158047-3",
            "preflabel": { "de": "Granulozyt" }
          },{
            "uri": "http://d-nb.info/gnd/4227943-4",
            "preflabel": { "de": "Leukozytenadhäsion" }
          },{
            "uri": "http://d-nb.info/gnd/4166696-3",
            "preflabel": { "de": "Leukozytenphosphatase" }
          },{
            "uri": "http://d-nb.info/gnd/4285013-7",
            "prefLabel": { "de": "Monozyt" }
        } ],
        "coordination": "OR"
    },
    "mappingRelevance": 0.5
}
```

```json
{
    "from": {
        "inScheme": ["http://dewey.info/scheme/edition/e22/"],    
        "conceptSet": [ {
            "uri": "http://dewey.info/class/612.112/e22/2014-04-15/",
            "notation": ["612.112"]
        } ]
    },
    "to": {
        "inScheme": ["http://d-nb.info/gnd/7749153-1"],
        "conceptSet": [ {
            "uri": "http://d-nb.info/gnd/4499720-6",
            "prefLabel": { "de": "Leukozytenintegrine" }
        } ]
    } 
}
```
</div>



----

This version: <http://gbv.github.io/jskos/{CURRENT_VERSION}.html> ({CURRENT_TIMESTAMP})\
Latest version: <http://gbv.github.io/jskos/> 

Created with [makespec](http://jakobib.github.io/makespec/)
