# Example: medium

Question: "I changed my site's A record an hour ago. Some people still reach the old server. Is that DNS propagation?"

## Response

The delay comes from caching, and the TTL of the old record, set before your change, fixes its length.

Two kinds of DNS server take part. An authoritative server is a DNS server that stores the records of a domain and answers queries from those records. A recursive resolver is a DNS server that receives a query from a client, queries authoritative servers on the client's behalf, returns the answer, and stores it in a cache. A client is a device or program that sends DNS queries, such as a laptop or a browser. Your change replaced the A record, the record that maps a domain name to an IPv4 address, on the authoritative servers. An IPv4 address is a 32-bit address written as four numbers from 0 to 255, such as 192.0.2.10. Authoritative servers send recursive resolvers no notice of a change.

Each record carries a TTL (time to live): the number of seconds a recursive resolver may keep the record in its cache. RFC 1035, section 4.1.3, defines the field as "a 32 bit unsigned integer that specifies the time interval (in seconds) that the resource record may be cached before it should be discarded." Resource record is the RFC's name for a record. A recursive resolver that fetched your old A record before the change answers from that cached copy until the TTL of the copy reaches 0. Then it queries an authoritative server again and receives the new address. The record changed at the source at one moment, while clients of a recursive resolver with an unexpired copy still receive the old address.

Example: the old record had TTL 3600. One recursive resolver fetched it at 09:50, and you changed the record at 10:00. That recursive resolver returns the old address until 10:50. A recursive resolver that fetched at 09:01 switches at 10:01. Lowering the TTL after the change does not shorten the wait, because each recursive resolver holds the old record with the old TTL.

You can observe both servers with dig, a command-line tool that sends one DNS query and prints the answer. I did not run these commands for this answer. The output lines show their form, and 192.0.2.10 is a documentation address.

- Remaining cache time at your recursive resolver: run `dig +noall +answer www.example.com A` twice, ten seconds apart. Each run prints a line such as `www.example.com. 1742 IN A 192.0.2.10`. The second field is the remaining TTL in seconds. A number about ten lower on the second run means the recursive resolver answered from its cache.
- Current value at the source: an NS record is a record whose data is the name of an authoritative server for a domain. Run `dig +short NS example.com` to list those names, then `dig www.example.com A @ns1.example.net` with one of them. The flags line reads `flags: qr aa rd`. The `aa` flag, short for authoritative answer, is a bit in a DNS reply that marks a reply from an authoritative server. That reply carries the new address and the full TTL you configured.

The old TTL bounds the wait for recursive resolvers that respect the TTL. An operator can configure a recursive resolver to keep records longer, and that recursive resolver returns the old address past the TTL. Operating systems and browsers keep caches of their own, which adds delay on the client after its recursive resolver has switched.

## Why it fits

Opens with the relationship between the change and the delay. Each entity gets an exact definition where it first appears. No analogy. One short numeric example. Each non-obvious point comes with the command or RFC section that proves it, and the response says the commands were not run. Limits stated.
