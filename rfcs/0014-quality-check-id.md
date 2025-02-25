# Quality Check Identifier

Champion: Simon Harrer

[Slack channel](https://data-mesh-learning.slack.com/archives/C08CN5BCBS5). Some early discussions were in the [WG channel](https://data-mesh-learning.slack.com/archives/C089S376YGM).

## Summary

## Motivation

TODO 

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

Add an `id` field of type `string` defined as the term [identifier](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/identifier) in Dublin Core Vocabulary as shown in the following table.

| Key                                  | UX label                  | Required | Description                                                                                                                                                                                |
|--------------------------------------|---------------------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|id                                    |Identifier                 |Yes       |Recommended practice is to identify the resource by means of a string conforming to an identification system. Examples include International Standard Book Number (ISBN), Digital Object Identifier (DOI), and Uniform Resource Name (URN). *Persistent identifiers SHOULD be provided as HTTP URI*s.

. We can also consider suggesting a preferred identification system, such as:

- HTTP URI ([RFC-3986](https://datatracker.ietf.org/doc/html/rfc3986)) as recommended by W3C ([Linked Data](https://www.w3.org/wiki/LinkedData))
- UUID ([RFC-4122](https://www.rfc-editor.org/rfc/rfc4122.html))

while leaving the final decision up to the user. 

**Pros:**

- Freedom in selecting the identification system: Users can choose the best system for their needs.
- Simple and easy to understand: It’s a straightforward solution that doesn’t require much complexity, making it user-friendly.

**Cons:**

- Risk of conflicting identifiers: Without clear governance policies, the same identifier might be used for different things, causing confusion both between and within organizations.
- Increased complexity in tooling: The variety of identification systems can complicate the development and maintenance of tools, as they need to support multiple formats.

**Example: using uuid as id**

```yaml
quality:
  - type: sql 
    query: |
      SELECT COUNT(*) FROM ${object} WHERE ${property} IS NOT NULL
    mustBeLessThan: 3600
    id: 160512f7-358b-487d-8d26-5222534264e4
```

**Example: using a HTTP URI as id**

```yaml
quality:
  - type: sql 
    query: |
      SELECT COUNT(*) FROM ${object} WHERE ${property} IS NOT NULL
    mustBeLessThan: 3600
    id: http://bitol.org/contracts/myContract/1.0.0/schemas/lastName/checks/missingValues/1.0.1
```

### Add `name` and `fqn` field

Aggiungere il campo nome come idetificatore di un quality check associato ad una proprietà dello scheama. Lo stesso nome può essere usato per definire check associati a differenti elementi dello schema. L'identificativo univoco del check è costruito a partire dal nome nel segunete mode:

```
urn:odcs:contracts:{contract-name}:{contract-major-version}:schema:{json-pointer-to element}:checks:{check-name}
```

ad esempio per un check di quality il cui name è notNull definito sulla proprietà lastName dello schema nella version 1.0.0 di un contract chiamato myContract il fully qualified name assumerebbe la seguente forma

```
urn:odcs:contracts:myContract:1:schema:lastName:checks:notNull
```

il fully qualified name potrebbe essere associato ad un campo opzionale chiamato fqn di tipo readOnly. Un field è readOnly se non deve essere specificato dall'utente ma può essere generato in modo univoco a partire da altri campi e usato per interagire con i tool esterni.

| Key                                  | UX label                  | Required | Description                                                         |
|--------------------------------------|---------------------------|----------|---------------------------------------------------------------------|
|name                                    |Name                 |Yes       | The name of the quality check. It MUST be unique within the quality checks defined on a schema entity. It's RECOMMENDED to use a camel case formatted string.
|fqn                                    |Name                 |Yes       |The unique universal idetifier of the quality check. It MUST be a URN of the form `urn:odcs:contracts:{contract-name}:{contract-major-version}:schema:{json-pointer-to element}:checks:{check-name}`. 

In questo scenario si può valutare anche l'aggiunte dei seguenti campi

- version: la versione del check utile se vogliamo tenere traccia anche delle precedenti versioni di uno stesso check
- displayName: poiche name è parte del fqn usato per identificare il check non può essere modificato senza modificare anche l'identità del check stesso. Potrebbe pertanto aver senso introdurre un campo displayName che indica il nome del check che si vuole usare a livello di UX e che può liberamente cambiare nel tempo
- id:  It's an UUID version 5 (RFC-4122) generated as SHA-1 hash of the fqn. Anche questo è un campo readOnly. l'fqn è di più facile comprensione per un utente metre l'id che da esso è genrato è di più semplice utilizzo nelle chiamate ad api esposte da tool esterni per identificare il check

 **Pros:**

1. **Uniqueness and Clear Identification:**
   - The use of a **fully qualified name (FQN)** ensures that each quality check has a globally unique identifier based on a clear, structured format. This eliminates ambiguity and prevents conflicts across contracts and versions.

2. **Structured and Scalable:**
   - The format `urn:odcs:contracts:{contract-name}:{contract-major-version}:schema:{json-pointer-to-element}:checks:{check-name}` provides a clear, hierarchical structure that is easy to expand or adapt as more checks are added or new contracts are introduced.

3. **Easy Integration with External Tools:**
   - By using **URN** as a unique identifier and adding an **UUID** (generated from the FQN) for easier integration with APIs, the solution is well-suited for external tool interaction. The UUID is easier to handle in API calls compared to the full FQN.

4. **Read-Only Fields:**
   - The `fqn` and `id` are **read-only**, making them automatically generated and ensuring consistency. This also minimizes the chances for errors during manual data entry.

5. **Version Control:**
   - The `version` field allows you to track different versions of the same quality check, which is useful for maintaining historical data and making backward-compatible improvements.

6. **User-Friendly Display Name:**
   - The **`displayName`** field allows for flexibility in the user experience (UX), as it enables the check's name to be modified for clarity and presentation without changing its underlying identity.

**Cons:**

1. **Complexity in Understanding FQN:**
   - The **FQN format** might be difficult for non-technical users to understand, as it includes multiple components (e.g., contract name, version, JSON pointer). This could make debugging or data tracking harder for some users, especially those unfamiliar with URN structures.

2. **Increased Schema Complexity:**
   - Adding multiple fields such as `version`, `displayName`, `fqn`, and `id` can make the schema more complex and increase the maintenance effort. This might create additional overhead for developers managing the schema and quality checks.

3. **Dependency on Governance for Consistency:**
   - The solution heavily relies on **clear governance** to ensure that the names and versions of quality checks are consistent. Without governance, the risk of inconsistent naming or versioning increases, potentially leading to confusion.

4. **Harder to Modify or Refactor:**
   - Since the **name** is embedded in the FQN and the `fqn` is used as the key identifier, any changes to the name would require modifications to the FQN and potentially the identifiers of dependent systems. This can make it challenging to modify quality checks without affecting many parts of the system.

5. **Limited Flexibility with FQN Changes:**
   - If a change to the FQN is necessary (e.g., if the schema structure or version changes), it would require updates to both the **`fqn`** and potentially any external tools or integrations that rely on it. This can cause issues with backward compatibility or data migration.

6. **Tooling and Infrastructure Overhead:**
   - Supporting UUID generation, handling multiple fields, and ensuring that all tools and systems interact correctly with the FQN might require additional infrastructure or tooling overhead, especially for maintaining backward compatibility when new versions of checks are introduced.

**Suggestions:**
- To mitigate the **complexity** for users, consider providing documentation or tooltips explaining the structure of the FQN.
- **Automated validation and governance** around naming conventions and versioning could help prevent inconsistency and reduce the risk of errors.
- Depending on the use case, you could consider an **alternate, simpler identifier format** for users who don’t need the full granularity of FQN but still require uniqueness.

This solution brings clarity and flexibility, but it also introduces potential complexity that needs careful management.


## Decision

TBD

## Consequences

TBD

## References

- TBD
