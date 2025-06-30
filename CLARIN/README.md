# Vocabularies and mappings for the [CLARIN VLO](https://vlo.clarin.eu)

For a demonstration of step-by-step retrieval of the data, see the notebook
[clarin_data.ipynb](./clarin_data.ipynb).

The retrieved data (concept definitions, profiles specifications, VLO facet values, 
concept - facet mapping definitions), can be found in the [data folder](./data).

## Context 

### Components

* Virtual Language Observatory (VLO)
  * Solr index
  * [Web UI](https://vlo.clarin.eu)
  * API (work in progress -> [alpha deployment](https://vlo.clarin-dev.eu/api))
  * Importer _processes harvested CMDI files into VLO records_
* Component Registry _stores and disseminates CMDI component and profile specifications_
  * Relational database
  * [Web UI](https://catalog.clarin.eu/ds/ComponentRegistry/)
  * [API](https://catalog.clarin.eu/ds/ComponentRegistry/api-docs/)
* Concept Registry _stores and disseminates concepts_
  * Triple store
  * Skosmos ([API](https://vocabularies.clarin.eu/clavas/rest/v1/ccr/) + [UI](https://concepts.clarin.eu))

### Overview of semantic artefacts and relations

```                                                                                                                                                                                            
    +-------+                   +----------------------+                 +----------------------+
    |Concept|<--<concept link>--| Element or attribute |--<defined in>-->|CMDI profile/component|
    +-------+                   +----------------------+                 +----------------------+
        |                                  ^                                                      
    <maps to>                              |                                                      
        |                           <value scheme for>                                            
        v                                  |                                                      
   +---------+                    +---------------------+                                         
   |VLO Facet|                    |Controlled vocabulary|                                         
   +---------+                    +---------------------+                                         
        |                                  |                                                      
 <contains value>                   <contains value>                                              
        |                                  |                                                      
        v                                  v                                                      
 +---------------+                    +--------+                                                  
 |VLO facet value|                    |CV value|                                                  
 +---------------+                    +--------+                                                  
        |                                  |                                                      
        |                                  v                                                      
        +--------> [Label]              [Label]                                                   
```

## Vocabularies (and pseudo-vocabularies)

### Concepts

* The CLARIN Concept Registry (CCR) is implemented as a vocabulary in Skosmos instance
at [concepts.clarin.eu](https://concepts.clarin.eu). Each concept is identified by a
handle URI that resolves (and self-redirects) to the registry.

The Skosmos API can be used to access the concepts in the registry:
* The CCR vocabulary: [/rest/v1/ccr/](https://vocabularies.clarin.eu/clavas/rest/v1/ccr/)
* Concept schemes are listed in the response for the above.
  Concepts can be retrieved per concept scheme, for instance:
  [/rest/v1/ccr/data?uri=http://hdl.handle.net/11459/CCR_P-Metadata_6f3f84d1-6f06-6291-4e20-4cd361cca128](https://vocabularies.clarin.eu/clavas/rest/v1/ccr/data?uri=http://hdl.handle.net/11459/CCR_P-Metadata_6f3f84d1-6f06-6291-4e20-4cd361cca128)

### Controlled vocabularies from profile specifications

* Via the Component Registry API (base URL: `https://catalog.clarin.eu/ds/ComponentRegistry/rest`, [documentation](https://catalog.clarin.eu/ds/ComponentRegistry/api-docs/)):
  * all public profiles:
  [/registry/1.x/profiles](https://catalog.clarin.eu/ds/ComponentRegistry/rest/registry/1.x/profiles)
  * fully expanded specification for an individual profile, e.g.:
  [/registry/1.x/profiles/clarin.eu:cr1:p_1349361150699/xml](https://catalog.clarin.eu/ds/ComponentRegistry/rest/registry/1.x/profiles/clarin.eu:cr1:p_1349361150699/xml)
* We can extract a list of identifiers for all profiles encountered when populating the VLO
from harvested metadata. This can serve as a starting point for extracting relevant vocabularies
from the profile specifications.
  * `TODO`: include a list of profile IDs
  * `TOOD`: include a script to retrieve the profile specifications

#### Example of a controlled vocabulary definition

The following snippet is sourced from [profile clarin.eu:cr1:p_1349361150699 'resourceInfo'](https://catalog.clarin.eu/ds/ComponentRegistry/rest/registry/1.x/profiles/clarin.eu:cr1:p_1349361150699/xml)

```xml
<Element name="licence" ConceptLink="http://hdl.handle.net/11459/CCR_C-2457_45bbaa1a-7002-2ecd-ab9d-57a189f694a6" CardinalityMin="1" CardinalityMax="unbounded" cue:DisplayPriority="1">
    <ValueScheme>
        <Vocabulary>
            <enumeration>
                <item ConceptLink="" AppInfo="">CC-BY</item>
                <item ConceptLink="" AppInfo="">CC-BY-NC</item>
                <item ConceptLink="" AppInfo="">CC-BY-NC-ND</item>
                <item ConceptLink="" AppInfo="">CC-BY-NC-SA</item>
                <item ConceptLink="" AppInfo="">CC-BY-ND</item>
                <item ConceptLink="" AppInfo="">CC-BY-SA</item>
                <item ConceptLink="" AppInfo="">CC-ZERO</item>
                <item ConceptLink="" AppInfo="">MS-C-NoReD</item>
                <!-- remaining items omitted -->
            </enumeration>
        </Vocabulary>
    </ValueScheme>
</Element>
```

In this example we see the definition of an element named 'licence' with a concept link
to `http://hdl.handle.net/11459/CCR_C-2457_45bbaa1a-7002-2ecd-ab9d-57a189f694a6`
("[license](http://hdl.handle.net/11459/CCR_C-2457_45bbaa1a-7002-2ecd-ab9d-57a189f694a6)").
Note that we can find this concept mapped to the `availability` and `license` fields in the
VLO's concept - facet mapping (see below). Elements that contain an element `ValueScheme/Vocabulary/enumeration`
have a value scheme that is restricted to the enumerated values – in other words, a
controlled vocabulary is defined in-line for this element. 

The same may also be done for Attribute definitions, for example:
```xml
<Element name="identifier" ValueScheme="anyURI" CardinalityMin="1" CardinalityMax="unbounded" cue:DisplayPriority="1">
    <AttributeList>
        <Attribute name="type" ConceptLink="http://hdl.handle.net/11459/CCR_C-5354_a4e41aa9-b8ba-59ea-a34e-6d7179e393be">
            <ValueScheme>
                <Vocabulary>
                    <enumeration>
                        <item>DOI</item>
                        <item>handle</item>
                    </enumeration>
                </Vocabulary>
            </ValueScheme>
        </Attribute>
    </AttributeList>
</Element>
```

### VLO facets

* A list of facets with most occurring values can be obtained from the API:
[/facets](https://vlo.clarin-dev.eu/api/facets)
* All occurring values with value counts can be obtained by adding the field name to the
path. For example: [/facets/country](https://vlo.clarin-dev.eu/api/facets/country)
* A source of additional information about the VLO fields, which could also be used
as a starting point for processing of the facets, is the facets configuration file
[facetsConfiguration.xml](https://github.com/clarin-eric/VLO-mapping/blob/master/config/facetsConfiguration.xml).
Field configurations that contain an element `<displayAs>primaryFacet</displayAs>` or 
`<displayAs>secondaryFacet</displayAs>` are shown as facets in the VLO.

## Mappings

### Concept - facet mappings

A definition [facetConcepts.xml](https://github.com/clarin-eric/VLO-mapping/blob/master/mapping/facetConcepts.xml)
is used to populate fields in the VLO index on the basis of concept links that appear in 
the CMD profile specifications. In addition, it also supports mapping based on explicit
XPaths, and specifying blacklisting patterns (also XPath based).

__Usage__: the URIs for all concepts mapped to facet `$FACET` can be retrieved from
[facetConcepts.xml](https://github.com/clarin-eric/VLO-mapping/blob/master/mapping/facetConcepts.xml)
by evaluating the XPath: `/facetConcepts/facetConcept/[@name=$FACET]/concept`

### Value maps

The VLO offers a mechanism for value-to-value mapping for (post-hoc) curation purposes,
e.g. error correction and value harmonisation. These mappings may be defined within or
across fields. 

The compiled mappings (XML files derived from CSV definitions) can be found
in a folder [value-maps/dist](https://github.com/clarin-eric/VLO-mapping/tree/master/value-maps/dist)
of the VLO-mapping repository. A [master.xml](https://github.com/clarin-eric/VLO-mapping/blob/master/value-maps/dist/master.xml)
file is used by the VLO by first processing the XInclude directives and then applying
the value maps where `/value-mapping/origin-facet/@name` matches the field that was selected
for the processed value (i.e. after applying concept-facet mapping).  

Applying the map means inserting all `value-map/target-value-set/target-value` values in the 
specified field (`target-value/@facet`) in case of a matching `value-map/target-value-set/source-value`.
The source value and/or existing values in the field may be either retained or ignored depending
on the values of `@removeSourceValue` and `@overrideExistingValues` boolean attributes on
the `target-facet` element.

### Post-processing maps

Maps used for post-processing can be found in the folder
[uniform-maps](https://github.com/clarin-eric/VLO-mapping/tree/master/uniform-maps). The
exact application of these maps is defined in the logic of the VLO. However, they share a
common structure in which an eumeration of `mappings/mapping` occurs, each of which 
defines a `normalizedValue` and a number of `variants` that are to be mapped to the 
normalised value. Normally, the normalised value should occur in the relevant field in
the VLO, replacing the listed variants.

An example usage is the mapping of language names
to the default label in the ISO639-3 language codes vocabulary:
[LanguageNameVariantsMap.xml](https://github.com/clarin-eric/VLO-mapping/blob/master/uniform-maps/LanguageNameVariantsMap.xml).
