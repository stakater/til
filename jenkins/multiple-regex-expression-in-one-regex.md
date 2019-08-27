# Multiple Regex Expressions

This guide is regarding using multiple regexes in a single regex expression:


Template

```bash
((regx-1)|(regx-2)|(regx-3))

```

Example:

```bash
^(([0-9]+.[0-9]+.[0-9]+-develop-[0-9]+-SNAPSHOT)|([0-9]+.[0-9]+.[0-9]+-production-[0-9]+-SNAPSHOT))$
```

Matches:

1- 0.0.0-production-2-SNAPSHOT

2- 0.0.0-develop-2-SNAPSHOT
