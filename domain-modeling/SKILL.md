---
name: domain-modeling
description: Model a business domain into clear entities, value objects, aggregates, and bounded contexts using a shared ubiquitous language; use when starting a new system, untangling a tangled model, defining service boundaries, or when domain terms are ambiguous, overloaded, or inconsistent across code, docs, and conversations.
---

# Domain Modeling

Domain modeling turns fuzzy business requirements into a precise, shared model of concepts and their relationships. A good model reduces translation errors between stakeholders and code, clarifies ownership, and guides where to draw service and module boundaries. This skill favors clarity of language and explicit boundaries over premature abstraction.

## When to use this skill

- Starting a greenfield system or a major new subsystem.
- The same word means different things to different teams (e.g., "order", "account", "user").
- Business logic is scattered and duplicated across layers with no clear home.
- You need to split a monolith or decide where a new service's boundary lies.
- Stakeholders and engineers repeatedly talk past each other.

## Instructions

1. **Gather the language.** Interview domain experts and collect the nouns and verbs they actually use. Write them down verbatim; do not rename them to "cleaner" technical terms.
2. **Build a glossary (ubiquitous language).** For each term, write a one-line definition, note synonyms to avoid, and flag terms that mean different things in different contexts.
3. **Identify entities vs. value objects.** An entity has a stable identity over time (a `Customer`, an `Order`). A value object is defined only by its attributes and is immutable (a `Money`, an `Address`, a `DateRange`).
4. **Group into aggregates.** Cluster entities/value objects that must change together under one consistency boundary. Pick an aggregate root as the only entry point; external references point to the root by ID.
5. **Draw bounded contexts.** Group aggregates by the language and rules that apply. A term that changes meaning marks a context boundary. Name each context.
6. **Map relationships between contexts.** For each pair, note the integration style (shared kernel, customer/supplier, conformist, anti-corruption layer) and the direction of dependency.
7. **Define invariants.** For each aggregate, list the rules that must always hold (e.g., "an order total equals the sum of its line items"). These belong inside the aggregate.
8. **Model behavior, not just data.** Attach the operations (verbs) to the entities that own them. Avoid anemic models where all logic sits in service classes.
9. **Validate with scenarios.** Walk concrete use cases through the model out loud with a domain expert. Adjust names and boundaries where the story doesn't flow.
10. **Keep it living.** Store the glossary and context map near the code and update them as the language evolves.

## Examples

A glossary snippet and aggregate sketch for an e-commerce domain:

```
Ubiquitous Language
- Cart:      items a shopper intends to buy; mutable; exists before checkout.
- Order:     a confirmed, immutable record of a purchase. NOT the same as Cart.
- Line Item: a product + quantity + captured price at time of order.
- Fulfillment: the shipping/delivery process; a SEPARATE bounded context.

Aggregate: Order (root)
  invariants:
    - total == sum(lineItem.price * lineItem.quantity)
    - status transitions: PLACED -> PAID -> SHIPPED -> DELIVERED | CANCELLED
  contains: LineItem[] (value objects), ShippingAddress (value object)
  references: customerId (by ID, not object)
```

Context map:

```
[Sales Context] --customer/supplier--> [Fulfillment Context]
[Sales Context] --anti-corruption layer--> [Legacy Billing]
```

## Checklist

- [ ] A written glossary exists and uses the experts' real words.
- [ ] Every term with multiple meanings is tied to a specific bounded context.
- [ ] Entities and value objects are clearly distinguished.
- [ ] Each aggregate has a single root and explicit invariants.
- [ ] Cross-aggregate references are by ID, not by embedded objects.
- [ ] A context map shows dependencies and integration styles.
- [ ] The model has been walked through real scenarios with a domain expert.
- [ ] Behavior lives with the data it governs (no anemic model).
