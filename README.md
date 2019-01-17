# rasa_composite_entities

A Rasa NLU component for composite entities.

## Installation

```
pip install rasa_composite_entities
```

The only external dependency is Rasa NLU itself, which should be installed anyway when you want to use this component.

After installation, the component can be added your pipeline like any other component:

```yaml
language: "en_core_web_md"

pipeline:
- name: "nlp_spacy"
- name: "tokenizer_spacy"
- name: "intent_featurizer_spacy"
- name: "intent_entity_featurizer_regex"
- name: "ner_crf"
- name: "ner_synonyms"
- name: "intent_classifier_sklearn"
- name: "rasa_composite_entities.CompositeEntityExtractor"
```

## Usage

Simply add another entry to your training file (in JSON format) defining composite patterns:
```json
"composite_entities": [
  {
    "name": "product_with_attributes",
    "patterns": [
      "@color @product with @pattern",
      "@pattern @color @product"
    ]
  }
],
"common_examples": [
    ...
]
```
Every word starting with a "@" will be considered a placeholder for an entity with that name. The component is agnostic to the origin of entities, you can use anything that Rasa NLU returns as the "entity" field in its messages. This means that you can not only use the entities defined in your common examples, but also numerical entities from duckling etc.

## Example

Composite entities act as containers that group several entities into logical units. Consider the following example phrase:
```
I am looking for a red shirt with stripes and checkered blue shoes.
```
Properly trained, Rasa NLU could return entities like this:
```json
"entities" : [
  {
    "start": 19,
    "end": 22,
    "value": "red",
    "entity": "color",
    "confidence": 0.8356,
    "extractor": "ner_crf"
  },
  {
    "start": 23,
    "end": 28,
    "value": "shirt",
    "entity": "product",
    "confidence": 0.9235,
    "extractor": "ner_crf"
  },
  {
    "start": 34,
    "end": 41,
    "value": "striped",
    "entity": "pattern",
    "confidence": 0.8962,
    "extractor": "ner_crf"
  },
  {
    "start": 46,
    "end": 55,
    "value": "checkered",
    "entity": "pattern",
    "confidence": 0.9311,
    "extractor": "ner_crf"
  },
  {
    "start": 56,
    "end": 60,
    "value": "blue",
    "entity": "color",
    "confidence": 0.8161,
    "extractor": "ner_crf"
  },
  {
    "start": 61,
    "end": 66,
    "value": "shoe",
    "entity": "product",
    "confidence": 0.8023,
    "extractor": "ner_crf"
  }
]
```

It's hard to infer exactly what the user is looking for from this output alone. Is he looking for a striped and checkered shirt? Striped and checkered shoes? Or a striped shirt and checkered shoes?

By defining common patterns of entity combinations, we can automatically create entity groups. If we add the composite entity patterns as in the usage example above, the output will be changed to this: 
```json
"entities" : [
  {
    "entity": "product_with_attributes",
    "type": "composite",
    "contained_entities": [
      {
        "start": 19,
        "end": 22,
        "value": "red",
        "entity": "color",
        "confidence": 0.8356,
        "extractor": "ner_crf",
        "type": "basic"
      },
      {
        "start": 23,
        "end": 28,
        "value": "shirt",
        "entity": "product",
        "confidence": 0.9235,
        "extractor": "ner_crf",
        "type": "basic"
      },
      {
        "start": 34,
        "end": 41,
        "value": "striped",
        "entity": "pattern",
        "confidence": 0.8962,
        "extractor": "ner_crf",
        "type": "basic"
      },
    ]
  },
  {
    "entity": "product_with_attributes",
    "type": "composite",
    "contained_entities": [
      {
        "start": 46,
        "end": 55,
        "value": "checkered",
        "entity": "pattern",
        "confidence": 0.9311,
        "extractor": "ner_crf",
        "type": "basic"
      },
      {
        "start": 56,
        "end": 60,
        "value": "blue",
        "entity": "color",
        "confidence": 0.8161,
        "extractor": "ner_crf",
        "type": "basic"
      },
      {
        "start": 61,
        "end": 66,
        "value": "shoe",
        "entity": "product",
        "confidence": 0.8023,
        "extractor": "ner_crf",
        "type": "basic"
      }
    ]
  }
]
```

