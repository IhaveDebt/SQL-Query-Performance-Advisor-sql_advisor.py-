
---

# 9 — SQL Query Performance Advisor (sql_advisor.py)

**File:** `src/sql_advisor.py`
```python
#!/usr/bin/env python3
"""
SQL Query Performance Advisor - sql_advisor.py

- Parses simple SELECT queries and offers optimization hints (index candidates, rewrite tips)
- This is heuristic-based and safe — does not execute queries
"""

import re

def parse_select(q):
    q = q.strip().lower()
    m = re.match(r"select\s+(?P<cols>[\*\w, ]+)\s+from\s+(?P<table>\w+)(?:\s+where\s+(?P<where>.+))?", q)
    if not m:
        return None
    return m.groupdict()

def advise(q):
    p = parse_select(q)
    if not p:
        return ["Could not parse query."]
    adv = []
    if "*" in p["cols"]:
        adv.append("Avoid SELECT * — list only required columns to reduce IO.")
    where = p.get("where")
    if where:
        # find equality predicates
        eqs = re.findall(r"(\w+)\s*=\s*['\w\d]+", where)
        for c in eqs:
            adv.append(f"Consider index on column `{c}` to speed up equality lookup.")
        if "like" in where:
            adv.append("LIKE predicates may prevent index usage; consider full-text index or trigram index.")
    else:
        adv.append("Query has no WHERE clause — may cause full table scan.")
    return adv

if __name__ == "__main__":
    examples = [
        "SELECT * FROM users WHERE email = 'a@b.com'",
        "SELECT id, name FROM orders WHERE created_at > '2024-01-01'",
        "SELECT * FROM products"
    ]
    for ex in examples:
        print("Query:", ex)
        for a in advise(ex):
            print(" -", a)
        print()
