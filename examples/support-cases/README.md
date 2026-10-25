# Synthetic Support Case fixtures

These are hand-authored design/validation examples, not measured model results or customer data. All people, organizations and conversations are fictional; email domains use example.com/example.org.

| Input | Expected output | Purpose |
| --- | --- | --- |
| [source-a.csv](source-a.csv) | [expected-a.json](expected-a.json) | Two support tickets; Open -> In Progress; missing optional email; multiline comments with and without metadata. |
| [source-b.csv](source-b.csv) | [expected-b.json](expected-b.json) | Two tickets in an alternative support export layout; different names and enums; timezone normalization; zero versus missing impact count. |
| [source-a-mixed.csv](source-a-mixed.csv) | [expected-mixed.json](expected-mixed.json) | One valid record and five independently invalid records; partial-on and all-or-nothing outcomes. |

The two layouts represent varying support source formats mapping into the same CRM-shaped Support Case contract. CRM is not a second source in this scenario. All inputs are flat CSV; related organization/reporter data is already assembled, and comments are one optional preassembled text cell. CSV quoting preserves embedded commas and newlines. Count parsed records, not physical lines, for the upload limit.

[fixture-mappings.json](fixture-mappings.json) records target-to-source column correspondences and explicit synthetic source enum meanings. These are expected answers for repeatable experiments, not a deployed mapping format, automatic approval, or a demand to hard-code AI responses. Priority translations in these fixtures follow their declared conventions. Real source semantics must be inferred/reviewed separately.

Expected records use UTC timestamps, explicit nulls for blank optional cells, integer counts and CRLF/CR-to-LF comment normalization. All preserve supplied comment order and wording. Other formatting operations remain to be bounded and reviewed.

An extra column demonstrates unmapped-field visibility. A supplied invalid optional email or impact count rejects that record. Missing optional email is valid. The mixed file has an unknown priority; a map that covers other defined values must not silently guess a value for it. The runtime handling and review of unseen values remain subject to the contract.

For all-or-nothing delivery, none of the six records is delivered, including the individually valid row. Five records fail validation; the exact UI classification of the otherwise valid row is not prescribed. The essential guarantee is zero stored canonical results and a failed-run audit. Original source records remain available under recovery/export and retention policies.

Validate targets with [support-case-v1.schema.json](../../schemas/support-case-v1.schema.json) using Draft 2020-12 plus format checking. This does not validate approval, isolation, AI quality or the full processing implementation. The examples establish expected behavior for later implementation and AI feasibility experiments.
