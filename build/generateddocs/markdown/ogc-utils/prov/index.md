
# Provenance Chain (Schema)

`ogc.ogc-utils.prov` *v0.1*

Schema for a provenance chain based on PROV vocabulary semantics, Agents, Activities and Entities. This schema is designed as a mix-in that can be used to add properties to other objects in a polymorphic way.

[*Maturity*](https://github.com/cportele/ogcapi-building-blocks#building-block-maturity): Development

## Description

## Provenance chain

A schema defining objects that may be referenced or nested as a chain of activities.

This schema implements the PROV vocabulary semantics.


## Examples

### Example Topology object
See panel to right - note that a more user friendly "collapsable" version is in development. 
#### json
```json
{ "@context": {
    "@base": "https://example.org/"},
  "id": "DP-1",
  "type": "Entity",

  "wasGeneratedBy": [
    "surveyreg-nz:DP-1-S1",
    {
      "id": "surveyreg-nz:DP-1-S2",
      "endedAtTime": "2029-01-01",
      "wasAssociatedWith": "linz-registered-surveyors:bc-3",
      "used": {
        "wasAttributedTo": "icsm-jurisdictions:nz",
        "link": {
          "href": "https://some.gov/linktoact/",
          "rel": "related"
        }
      }
    }
  ],
  "provenance":
  [
    {
      "id": "DP-2223",
      "type": "Entity",
      "wasGeneratedBy": {
        "type": "Activity",
        "id": "surveyreg-nz:DP-1-S1",
        "endedAtTime": "2023-10-05",
        "wasAssociatedWith": "linz-registered-surveyors:ah-2344503",
        "used": {
          "id": "Act3",
          "type": "Entity",
          "wasAttributedTo": "icsm-jurisdictions:nz",
          "link": {
            "href": "https://some.gov/linktoact/",
            "rel": "related"
          }
        }
      }
    },
    {
      "id": "survprov-nz:survey",
      "type": "Activity",
      "endedAtTime": "2023-10-05"
    }
  ]
}




```

## Schema

```yaml
$schema: https://json-schema.org/draft/2020-12/schema
description: provenance chain
$defs:
  oneOrMoreActivitiesOrRefIds:
    oneOf:
    - type: string
    - $ref: '#/$defs/Activity'
    - type: array
      items:
        anyOf:
        - type: string
        - $ref: '#/$defs/Activity'
  oneOrMoreEntitiesOrRefIds:
    oneOf:
    - type: string
    - $ref: '#/$defs/Entity'
    - type: array
      items:
        anyOf:
        - type: string
        - $ref: '#/$defs/Entity'
  oneOrMoreAgentsOrRefIds:
    oneOf:
    - type: string
    - $ref: '#/$defs/externalLink'
    - $ref: '#/$defs/Agent'
    - type: array
      items:
        anyOf:
        - type: string
        - $ref: '#/$defs/Agent'
  externalLink:
    $ref: https://beta.schemas.opengis.net/json-fg/link.json
  Entity:
    type: object
    additionalProperties: false
    properties:
      '@context':
        description: allow a local context to set URI bases for object Ids
        type: object
        x-jsonld-id: http://www.w3.org/ns/prov#@context
      id:
        type: string
        x-jsonld-id: http://www.w3.org/ns/prov#@id
      type:
        type: string
        const: Entity
        x-jsonld-id: http://www.w3.org/ns/prov#@type
      provenance:
        $ref: '#/$defs/Prov'
        x-jsonld-id: prov-x:provenance
        x-jsonld-type: http://www.w3.org/ns/prov#@id
        x-jsonld-container: '@set'
      wasGeneratedBy:
        $ref: '#/$defs/oneOrMoreActivitiesOrRefIds'
        x-jsonld-type: http://www.w3.org/ns/prov#@id
        x-jsonld-id: http://www.w3.org/ns/prov#wasGeneratedBy
      wasAttributedTo:
        $ref: '#/$defs/oneOrMoreAgentsOrRefIds'
        x-jsonld-id: http://www.w3.org/ns/prov#wasAttributedTo
      wasDerivedFrom:
        $ref: '#/$defs/oneOrMoreEntitiesOrRefIds'
        x-jsonld-type: http://www.w3.org/ns/prov#@id
        x-jsonld-id: http://www.w3.org/ns/prov#wasDerivedFrom
      link:
        $ref: '#/$defs/externalLink'
        x-jsonld-id: http://www.w3.org/ns/prov#link
    required:
    - id
    - type
    anyOf:
    - required:
      - wasGeneratedBy
    - required:
      - wasAttributedTo
    - required:
      - wasDerivedFrom
    - required:
      - provenance
  dateOrTime:
    oneOf:
    - type: string
      format: date
      pattern: ^\d{4}-\d{2}-\d{2}$
    - type: string
      format: date-time
      pattern: ^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(?:\.\d+)?(?:Z|[+-]\d{2}:\d{2})?$
  Activity:
    type: object
    properties:
      type:
        type: string
        const: Activity
        x-jsonld-id: http://www.w3.org/ns/prov#@type
      endedAtTime:
        $ref: '#/$defs/dateOrTime'
        x-jsonld-id: http://www.w3.org/ns/prov#endedAtTime
      wasAssociatedWith:
        $ref: '#/$defs/oneOrMoreAgentsOrRefIds'
        x-jsonld-type: http://www.w3.org/ns/prov#@id
        x-jsonld-id: http://www.w3.org/ns/prov#wasAssociatedWith
      wasInformedBy:
        $ref: '#/$defs/oneOrMoreActivitiesOrRefIds'
        x-jsonld-id: http://www.w3.org/ns/prov#wasInformedBy
      used:
        $ref: '#/$defs/oneOrMoreEntitiesOrRefIds'
        x-jsonld-type: http://www.w3.org/ns/prov#@id
        x-jsonld-id: http://www.w3.org/ns/prov#used
    required:
    - type
    anyOf:
    - required:
      - used
    - required:
      - wasInformedBy
    - required:
      - endedAtTime
    - required:
      - startedAtTime
    - required:
      - wasAssociatedWith
  Agent:
    type: object
    properties:
      registeredName:
        type: string
        x-jsonld-id: http://xmlns.com/foaf/0.1/name
      id:
        type: string
        format: uri
        x-jsonld-id: http://www.w3.org/ns/prov#@id
      actedOnBehalfOf:
        $ref: '#/$defs/oneOrMoreAgentsOrRefIds'
        x-jsonld-id: http://www.w3.org/ns/prov#actedOnBehalfOf
    required:
    - registeredName
  Prov:
    description: An list of provenance objects linked together to form a provenance
      chain for the current object. Objects may have nested objects as well, with
      or without referencable ids
    type: array
    items:
      oneOf:
      - $ref: '#/$defs/Entity'
      - $ref: '#/$defs/Activity'
      - $ref: '#/$defs/Agent'
anyOf:
- $ref: '#/$defs/Entity'
- $ref: '#/$defs/Activity'
x-jsonld-extra-terms:
  prov: http://www.w3.org/ns/prov#
  survtypes-nz: https://surveytypes-nz/
  surveyreg-nz: https://surveys-nz/
  links:
    x-jsonld-id: http://www.w3.org/2000/01/rdf-schema#seeAlso
    x-jsonld-context:
      href: http://www.w3.org/ns/prov#@id
      title: rdfs:label
x-jsonld-prefixes:
  foaf: http://xmlns.com/foaf/0.1/

```

Links to the schema:

* YAML version: [schema.yaml](https://raw.githubusercontent.com/ogcincubator/bblock-prov-schema/master/build/annotated/ogc-utils/prov/schema.json)
* JSON version: [schema.json](https://raw.githubusercontent.com/ogcincubator/bblock-prov-schema/master/build/annotated/ogc-utils/prov/schema.yaml)


# JSON-LD Context

```jsonld
{
  "@context": {
    "id": "http://www.w3.org/ns/prov#@id",
    "type": "http://www.w3.org/ns/prov#@type",
    "provenance": {
      "@id": "prov-x:provenance",
      "@type": "http://www.w3.org/ns/prov#@id",
      "@container": "@set",
      "@context": {
        "used": {
          "@type": "http://www.w3.org/ns/prov#@id",
          "@id": "http://www.w3.org/ns/prov#used"
        },
        "registeredName": "foaf:name",
        "actedOnBehalfOf": "http://www.w3.org/ns/prov#actedOnBehalfOf"
      }
    },
    "wasGeneratedBy": {
      "@type": "http://www.w3.org/ns/prov#@id",
      "@id": "http://www.w3.org/ns/prov#wasGeneratedBy",
      "@context": {
        "used": {
          "@type": "http://www.w3.org/ns/prov#@id",
          "@id": "http://www.w3.org/ns/prov#used"
        }
      }
    },
    "wasAttributedTo": {
      "@id": "http://www.w3.org/ns/prov#wasAttributedTo",
      "@context": {
        "registeredName": "foaf:name",
        "actedOnBehalfOf": "http://www.w3.org/ns/prov#actedOnBehalfOf"
      }
    },
    "wasDerivedFrom": {
      "@type": "http://www.w3.org/ns/prov#@id",
      "@id": "http://www.w3.org/ns/prov#wasDerivedFrom"
    },
    "link": "http://www.w3.org/ns/prov#link",
    "endedAtTime": "http://www.w3.org/ns/prov#endedAtTime",
    "wasAssociatedWith": {
      "@type": "http://www.w3.org/ns/prov#@id",
      "@id": "http://www.w3.org/ns/prov#wasAssociatedWith",
      "@context": {
        "registeredName": "foaf:name",
        "actedOnBehalfOf": "http://www.w3.org/ns/prov#actedOnBehalfOf"
      }
    },
    "wasInformedBy": "http://www.w3.org/ns/prov#wasInformedBy",
    "used": {
      "@type": "http://www.w3.org/ns/prov#@id",
      "@id": "http://www.w3.org/ns/prov#used",
      "@context": {
        "provenance": {
          "@id": "prov-x:provenance",
          "@type": "http://www.w3.org/ns/prov#@id",
          "@container": "@set",
          "@context": {
            "registeredName": "foaf:name",
            "actedOnBehalfOf": "http://www.w3.org/ns/prov#actedOnBehalfOf"
          }
        },
        "wasGeneratedBy": {
          "@type": "http://www.w3.org/ns/prov#@id",
          "@id": "http://www.w3.org/ns/prov#wasGeneratedBy"
        }
      }
    },
    "prov": "http://www.w3.org/ns/prov#",
    "survtypes-nz": "https://surveytypes-nz/",
    "surveyreg-nz": "https://surveys-nz/",
    "links": {
      "@id": "http://www.w3.org/2000/01/rdf-schema#seeAlso",
      "@context": {
        "href": "http://www.w3.org/ns/prov#@id",
        "title": "rdfs:label"
      }
    },
    "foaf": "http://xmlns.com/foaf/0.1/"
  }
}
```

You can find the full JSON-LD context here:
[context.jsonld](https://raw.githubusercontent.com/ogcincubator/bblock-prov-schema/master/build/annotated/ogc-utils/prov/context.jsonld)


# For developers

The source code for this Building Block can be found in the following repository:

* URL: [https://github.com/ogcincubator/bblock-prov-schema](https://github.com/ogcincubator/bblock-prov-schema)
* Path: `_sources`
