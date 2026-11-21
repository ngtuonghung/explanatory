# explanatory

Agent skill that explains a subject at one of three levels.

```
/explanatory [low|medium|high] <subject>
```

- `low`: verdict plus the one missing connection, in a few sentences. Uses the `caveman` skill for voice.
- `medium` (default): exact, detailed teaching that connects what you already know. Inline definitions, quick examples, evidence for each non-obvious point.
- `high`: patient ground-up teaching. Glossary table first, analogies with their limits, worked examples, evidence.

All levels apply the writing rules of [stop-slop](https://github.com/hardikpandya/stop-slop) by Hardik Pandya (MIT, see `references/LICENSE-stop-slop`).
