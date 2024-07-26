[source](https://openreview.net/pdf?id=WDQ9ZzsgDL)

# Introduction

Three Prerequisites
1. the entity definitions
2. the few-shot from the target domain
3. a set of explanations justifying the applicability of each entity tag

can be fully automated for two steps:
1. the crafting of the definition (simply copy-paste from documents itself)
2. the explanations for the few shot examples

# Method

Example:
```text
Defn: An entity is a person(person), university(university), scientist(scientist), organisation(organisation),
country(country), location(location), scientific discipline(discipline), enzyme(enzyme),
protein(protein), chemical compound(chemicalcompound), chemical element(chemicalelement),
event(event), astronomical object(astronomicalobject), academic journal(academicjournal),
award(award), or theory(theory). If an entity does not fit the types above it is (misc)

Example 1: He attended the U.S. Air Force Institute of Technology for a year , earning a bachelor 's
degree in aeromechanics , and received his test pilot training at Edwards Air Force Base in California
before his assignment as a test pilot at Wright-Patterson Air Force Base in Ohio .

Answer:
1. U.S. Air Force Institute of Technology | True | as he attended this institute is likely a university
(university)
2. bachelor 's degree | False | as it is not a university, award or any other entity type
3. aeromechanics | True | as it is a scientific discipline (discipline)
4. Edwards Air Force Base | True | as an Air Force Base is an organised unit (organisation)
5. California | True | as in this case California refers to the state of California itself (location)
6. Wright-Patterson Air Force Base | True | as an Air Force Base is an organisation (organisation)
7. Ohio | True | as it is a state (location)
```

3 key components:
1. Conditional Generation Language Model
2. Modular Definitions
3. Potential Entity Output Template

# Ablations and Investigations

## Components of PromptNER

- definition
- few-shot examples
- requirement to explain the reasoning on why a candidate is or is not an entity
- whether or not the list should contain only entities, or also contain candidate entities that may be declared as not meeting the definition of an entity

these 4 components are all make a big difference

## Difference ub Annotation
the explanation/annotation generated automatically by LLM results a significantly worse performance compares to human annotation is used
 

