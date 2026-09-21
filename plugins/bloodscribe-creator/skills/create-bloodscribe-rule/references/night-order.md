# Night order sections

Require the catalog feature `nightOrderSections`. Every enabled wake uses `firstTiming` or `otherTiming` with a catalog `section` and integer `position` from 0 to 30. These are independent between the first night and later nights. Never invent a global rank or infer a section from a legacy number. Preserve mixed abilities within one wake.

A tale may only override `nightOrder.sections.first[section]` or `.other[section]` with a permutation of IDs belonging to that exact section. A principal section and its exception are different boundaries. More than 31 members are allowed. Reset by removing the override; preserve remaining relative order when changing the cast and place additions at the latest legal position.

Declare required `before: [id]` and `immediatelyAfter: id` relationships. References apply when both entries are present, must remain compatible with the timeline, and must not form cycles. Mandatory dependencies beat user changes and random ties. Ties are randomized once per game and stored in its content snapshot. Guaranteed information remains unassigned until mechanics prove the guarantee; the label itself grants no truthfulness. Validate the complete pack, including rule wakes and additional private deliveries.
