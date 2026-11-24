# 11. FAQ & Known Limitations


**Q: Auto-detection picked the wrong dialect.**
A: Mixed-dialect scripts can be misdetected — specify the dialect manually. Confirm available dialects via `GET /api/dialects`.

**Q: Can dynamic SQL (`EXECUTE IMMEDIATE` / `sp_executesql`) be analyzed?**
A: SQL assembled at runtime cannot be expanded statically. Esther detects it, emits a `DYNAMIC_SQL` warning and marks affected nodes at 30% confidence — nothing is silently dropped.

**Q: My analysis-page results disappeared after refresh.**
A: The analysis page is an ad-hoc sandbox and does not persist. Use extraction pipelines for long-lived, cross-object lineage (Chapter 5).

**Q: Why must a SQL package in an extraction pipeline declare a dialect?**
A: Auto-detection over a whole package is prone to misjudgment, which would silently drop parses and assets. Script packages therefore require an explicit database type, double-checked at run time.

**Known limitations**

| Limitation | Details |
|------------|---------|
| Traversal depth | No hard cap (runs to full depth by default); for very large graphs, narrow the view with type filters / endpoint tracing |
| Auto-layout layers | At most 12 layers per graph; for very large graphs, browse per asset |
| `POST /api/analyze` unauthenticated | Keep it on internal networks |
| GOTO static uncertainty | DML skipped by GOTO is flagged with `GOTO_UNCERTAINTY` and conservatively marked |
| Antivirus false positives | Nuitka builds may trigger some antiviruses — allowlist the program |

---

## Purchasing & Licensing FAQ

The most common questions from our website visitors. For anything else, visit [shumai-data.com](https://shumai-data.com) and contact us.

**Q: How do I get a trial license?**

Register on our website [shumai-data.com](https://shumai-data.com) to receive an all-modules trial coupon. The license is issued automatically and can be downloaded from the console.

**Q: How long does a bank-transfer order take to take effect?**

Our finance team verifies payment within 1 business day of receipt; the order then becomes active and the license is generated automatically.

**Q: Can I get an invoice?**

VAT general and special invoices are supported. You can request an invoice from the console after payment.

**Q: What if I change computers?**

Console → License Management → Reset device binding. Self-service resets are allowed twice per calendar month.

**Q: Will I lose data when my license expires?**

No. Expiration only disables paid features; local data is fully preserved and restored immediately upon renewal.

**Q: Which deployment options are supported?**

Three delivery forms — Windows / Linux / Docker — all running fully offline; data never leaves your intranet.

