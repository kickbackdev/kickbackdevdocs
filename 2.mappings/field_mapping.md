### Partner

<img src="../images/logokick.avif" alt="Kickback logo" width="120" />

| Target_Field            | Source_System | Source_Field_or_Value    | Transformation_Logic_or_Step           | Notes_or_Path                       |
| ----------------------- | ------------- | ------------------------ | -------------------------------------- | ----------------------------------- |
| Partner Name            | SignWell      | signer_name              | (None)                                 | Common                              |
| Email                   | SignWell      | signer_email             | KKB-F1-4: FMT Email (lower+trim)       | Common (Unique Key)                 |
| Partner Type            | Static Value  | Individual               | (None)                                 | Path A: Individual Partner          |
| Partner Type            | Static Value  | Shop                     | (None)                                 | Path B: Shop Partner                |
| Partner Type            | Static Value  | Ambassador               | (None)                                 | Path C: Ambassador                  |
| Shop Type               | SignWell      | shop_type (custom_field) | KKB-F1-4: FMT shop_type (Lookup Table) | Path B: Shop Partner                |
| Charter Status End Date | SignWell      | completed_at             | KKB-F1-6: FMT Charter End Date (+1y)   | Path B: Shop Partner (Charter only) |

---

### Contracts

| Target_Field  | Source_System   | Source_Field_or_Value     | Transformation_Logic_or_Step     | Notes_or_Path                 |
| ------------- | --------------- | ------------------------- | -------------------------------- | ----------------------------- |
| Contract ID   | SignWell        | doc_id                    | (None)                           | Common (Unique Key, KKB-F1-9) |
| Partner       | Zapier Internal | AT Find/Create Partner.id | (Link Record)                    | Path A, B, C                  |
| Contract Type | Static Value    | Individual Partner        | (None)                           | Path A: Individual Partner    |
| Contract Type | Static Value    | Shop Partner              | (None)                           | Path B: Shop Partner          |
| Contract Type | Static Value    | Ambassador                | (None)                           | Path C: Ambassador            |
| Date Signed   | SignWell        | completed_at              | KKB-F1-4: FMT completed_at (UTC) | Common                        |
| Document Link | SignWell        | final_url                 | (None)                           | Common                        |

---

### Parts

| Target_Field          | Source_System   | Source_Field_or_Value              | Transformation_Logic_or_Step        | Notes_or_Path                                 |
| --------------------- | --------------- | ---------------------------------- | ----------------------------------- | --------------------------------------------- |
| Owner                 | Zapier Internal | AT Find Partner (Addendum).id      | (Link Record)                       | Path D: Parts Addendum                        |
| Part Description      | SignWell        | part_description (custom_field)    | (None)                              | Path D: Parts Addendum                        |
| Vehicle Compatibility | SignWell        | year, make, model (custom_fields)  | KKB-F1-8: CODE (Join strings)       | Path D: Parts Addendum                        |
| Condition             | SignWell        | condition (custom_field)           | KKB-F1-8: CODE (Normalize to As-Is) | Path D: Parts Addendum                        |
| Status                | Static Value    | Contract Signed - Awaiting Receipt | (None)                              | Path D: Parts Addendum                        |
| Date Added            | Zapier Internal | Now                                | (None)                              | Path D: Parts Addendum                        |
| Source Doc ID         | SignWell        | doc_id                             | (None)                              | Path D: Parts Addendum (Unique Key, KKB-F1-9) |

---

### F1_Logs

| Target_Field | Source_System   | Source_Field_or_Value       | Transformation_Logic_or_Step | Notes_or_Path             |
| ------------ | --------------- | --------------------------- | ---------------------------- | ------------------------- |
| Timestamp    | Zapier Internal | (Zap Run Time or Now)       | (None)                       | KKB-F1-13 (Error Logging) |
| Level        | Static Value    | info / warn / error         | (None)                       | KKB-F1-13 (Error Logging) |
| Message      | Zapier Internal | (Error Message or Log Text) | (None)                       | KKB-F1-13 (Error Logging) |
| Doc ID       | SignWell        | doc_id                      | (None)                       | KKB-F1-13 (Error Logging) |
| Email        | SignWell        | normalized_email            | (None)                       | KKB-F1-13 (Error Logging) |
| Zap Run ID   | Zapier Internal | (Zap Run ID)                | (None)                       | KKB-F1-13 (Error Logging) |
