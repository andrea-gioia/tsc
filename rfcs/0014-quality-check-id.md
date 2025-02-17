# Quality Check Identifier

Champion: Simon Harrer

[Slack channel](https://data-mesh-learning.slack.com/archives/C08CN5BCBS5). Some early discussions were in the [WG channel](https://data-mesh-learning.slack.com/archives/C089S376YGM).

## Summary

## Motivation

A quality check is hard to identify at the moment. 

```yaml
quality:
  - type: sql 
    query: |
      SELECT COUNT(*) FROM ${object} WHERE ${property} IS NOT NULL
    mustBeLessThan: 3600    
```

- We could use "name" (but can quickly become hard to stay unchanged),
- or use a custom property (not natively supported, harder to parse),
- or the path in the YAML (but this could change quickly when restructuring the YAML, resorting, ...).

But all are not good solutions.

## Design and examples

Add an optional ID field with type string. We can recommend using UUIDs, but should not restrict it to only them.

```yaml
quality:
  - type: sql 
    query: |
      SELECT COUNT(*) FROM ${object} WHERE ${property} IS NOT NULL
    mustBeLessThan: 3600
    id: 160512f7-358b-487d-8d26-5222534264e4
```

## Alternatives

### Add an `id` field

Add an `id` field of type `string` defined as the term [identifier](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/identifier) in Dublin Core Vocabulary. We can also evaluate to suggest some identification schema like UUID but leave to the user the final decision. This is the simplest solution for the user but in absence of clear governance policies opens the possibility of having conflicting identifiers among different contracts between organizations or even within the same organization. Moreover, this solution because of the many possible identification schemas used in practice makes the work of tooling more complex.

| Key                                  | UX label                  | Required | Description                                                                                                                                                                                |
|--------------------------------------|---------------------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|id                                    |Identifier                 |Yes       |Recommended practice is to identify the resource by means of a string conforming to an identification system. Examples include International Standard Book Number (ISBN), Digital Object Identifier (DOI), and Uniform Resource Name (URN). Persistent identifiers should be provided as HTTP URIs.

**Example: using uuid as id**

```yaml
quality:
  - type: sql 
    query: |
      SELECT COUNT(*) FROM ${object} WHERE ${property} IS NOT NULL
    mustBeLessThan: 3600
    id: 160512f7-358b-487d-8d26-5222534264e4
```
### Add `name` and `fqn` field

| Key                                  | UX label                  | Required | Description                                                                                                                                                                                |
|--------------------------------------|---------------------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|name                                    |Name                 |Yes       | The name of the quality check. It MUST be unique within the quality checks defined on a schema entity. It's RECOMMENDED to use a camel case formatted string.
|fqn                                    |Name                 |Yes       |The unique universal idetifier of the quality check. It MUST be a URN of the form urn:odcs:contracts:{product-name}:{contract-major-version}:schema:{json-pointer-to element}:checks:{check-name}. 

we can also add version 

| Key                                  | UX label                  | Required | Description                                                                                                                                                                                |
|--------------------------------------|---------------------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|name                                    |Name                 |Yes       |
|version                                    |Name                 |Yes       |
|fqn                                    |Name                 |Yes       |

finally we can suggest to use UUI as ID


## Decision

TBD

## Consequences

TBD

## References

- TBD
