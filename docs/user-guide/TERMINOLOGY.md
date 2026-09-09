# User-Facing Terminology

HomeMaker keeps precise domain names in backend code, API contracts, developer documentation, provenance, and advanced diagnostic detail. Everyday UI should prefer the following plain-language terms when the exact implementation term is not required.

| Domain / implementation term | Everyday UI term | Meaning |
| --- | --- | --- |
| Produced stock | Prepared food | Food produced by a planned Meal or Recipe and available for later use. |
| Recipe output | Prepared item | A prepared result from a Recipe that can be used later. |
| Cycle / plan validation | Check plan / Plan issues | Review the current Meal Cycle for blockers, shortages, conflicts, or warnings. |
| Reservation | Reserved for planned meals | Inventory quantity set aside for future planned demand; it is not consumed until completion/allocation records actual use. |
| Allocation preview | Planned ingredient use | A preview of which physical lots should be used first; it does not consume Inventory. |

## Usage rules

- Do not rename backend models, fields, schemas, API paths, query keys, or developer reference terminology solely to match UI copy.
- Preserve the distinction between food reserved for planned meals and food actually consumed.
- Preserve the distinction between physical Ingredient Inventory and prepared food.
- Technical/provenance views may use exact domain terms when precision is more useful than simplified copy.
- Avoid global search-and-replace. Choose wording based on the context of the task the user is completing.
