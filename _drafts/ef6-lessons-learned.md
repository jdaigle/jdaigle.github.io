1. Don't use many-to-many mappings. Instead, explicitly map the join table as an entity.

2. Disable Lazy Loading. Use eager fetching techniques.

3. Watch for cartesian products.

4. LINQ is great. But indecipherable LINQ is not smart. Use SQL when appropriate.

5. Use projections, particularly for reading and for view models.

6. Use batched/future queries (requires extension).

7. Avoid deferred execution.

8. Don't abstract EF behind repositories. Use commands/queries instead.

9. Explicit Transactions

10. 