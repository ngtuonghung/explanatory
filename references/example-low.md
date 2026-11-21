# Example: low

Question: "I changed my site's A record an hour ago. Some people still reach the old server. Is that DNS propagation?"

## Response

Partly right. No server pushes the change. Delay is cache expiry in recursive resolvers. TTL (time to live): number of seconds a recursive resolver may keep a record in its cache. Each recursive resolver keeps your old A record until the TTL of its copy runs out. Missing link: old TTL, set before your change, caps the wait. Lowering TTL now does not shorten it.

## Why it fits

Verdict first. Mechanism named, not walked through. TTL is the missing connection, so it gets one exact definition. One term per entity ("recursive resolver" every time). No glossary, analogy, or example.
