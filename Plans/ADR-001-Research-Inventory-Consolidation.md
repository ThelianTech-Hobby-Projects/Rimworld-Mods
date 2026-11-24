# ADR-001 — Consolidate Workshop research inventory

**Date:** 2026-11-23  
**Status:** Accepted — queue cleanup authorized; research discrepancies remain open.

## Context
The classification review was maintained in separate original and resolved queues, duplicating the main inventory.

## Decision
Consolidate available approved classifications, descriptions and feature tags into the Markdown catalog and enriched JSON, matched by Steam Workshop ID. Make these inventories authoritative for finalized research. Retire both review queues following explicit authorization; retain historical research, licensing evidence, relationship research and unavailable Workshop IDs.

## Outcome
- Processed 239 review positions covering 238 distinct IDs: 227 retained, seven transferred to secondary reference, and four scheduled for removal from active research.
- Active inventory decreased from 795 to 784; all 891 structured records remain preserved.
- Integration checks reported no duplicate active IDs and no unintended changes to unrelated records.
- 23 reconciliation items remain documented in `Integration-Discrepancies.md`; missing approved fields were not invented. The referenced expanded review was unavailable, so complete historical research verification remains outstanding.

## Consequences
The catalog and JSON are the maintained inventory; future corrections should update both. The integration report and discrepancy register remain as audit evidence. Queue retirement does not resolve outstanding research discrepancies.
