# Example: high

Question: "I changed my site's A record an hour ago. Some people still reach the old server. Is that DNS propagation?"

## Response

| Term | Definition |
|---|---|
| IP address | Number that identifies a device on the internet, so other devices can send data to it. |
| IPv4 address | IP address of 32 bits, written as four numbers from 0 to 255 joined by dots, for example 192.0.2.10. |
| Domain name | Name written as labels joined by dots, for example www.example.com. |
| Domain Name System (DNS) | Distributed system of servers that maps domain names to records. |
| Record | One entry in DNS. It holds a domain name, a type, and data, and carries a TTL. RFC 1035 calls it a resource record. |
| A record | Record whose data is the IPv4 address of a domain name. |
| NS record | Record whose data is the name of an authoritative server for a domain. |
| Client | Device or program that sends DNS queries, for example a laptop or a browser. |
| Authoritative server | DNS server that stores the records of a domain and answers queries from those records. |
| Recursive resolver | DNS server that receives a query from a client, queries authoritative servers on the client's behalf, returns the answer, and stores it in a cache. |
| Cache | Store of earlier answers that a recursive resolver keeps, so it can answer without a new query. |
| TTL (time to live) | Number of seconds, stored in each record, that a recursive resolver may keep the record in its cache. |
| dig | Command-line tool that sends one DNS query and prints the answer. |
| `aa` flag | Bit in a DNS reply, short for authoritative answer, that marks a reply from an authoritative server. |

Computers on the internet reach each other by numbers. An IP address is a number that identifies a device on the internet, so other devices can send data to it. People remember names better than numbers, so your site also has a domain name: a name written as labels joined by dots, for example www.example.com. The Domain Name System (DNS) connects the two. DNS is a distributed system of servers that maps domain names to records. A record is one entry in DNS. It holds a domain name, a type, and data, and carries a TTL, a lifetime in seconds. Your A record is the record whose data is the IPv4 address of your domain name, and an IPv4 address is an IP address of 32 bits, written as four numbers from 0 to 255 joined by dots, for example 192.0.2.10. You replaced that address. Some visitors still reach the old server because servers between them and your record keep copies of the old answer for a set time.

Picture two visitors. You change the address at 10:00. At 10:05, Ana opens your site on her laptop and reaches the new server. At the same minute, Ben, on a different internet provider, reaches the old server, and he keeps reaching it until 10:50. Both laptops asked for the same domain name and got different answers.

Picture a school office that keeps the official list of student phone numbers. Teachers do not walk to the office before each call. A teacher copies a number onto a sticky note, uses the note until the end of the day, and then throws it away. If a student's number changes at noon, the office list is correct at noon, and each teacher keeps calling the old number until that teacher's note goes in the bin.

In DNS terms, the office is the authoritative server: a DNS server that stores the records of a domain and answers queries from those records. Each teacher is a recursive resolver, usually run by an internet provider or a public service. A recursive resolver is a DNS server that receives a query from a client, queries authoritative servers on the client's behalf, returns the answer, and stores it in a cache. Ana's laptop and Ben's laptop are clients: a client is a device or program that sends DNS queries, for example a laptop or a browser. The sticky note is the cache, the store of earlier answers that a recursive resolver keeps, so it can answer without a new query. The end-of-day rule is the TTL (time to live): the number of seconds, stored in each record, that a recursive resolver may keep the record in its cache. The authoritative server sends recursive resolvers no notice of your change. Each recursive resolver answers from its cache until the TTL of its copy reaches 0, then queries the authoritative server again and receives the new address.

The school picture fails in two places. Every sticky note lasts until the end of the day, while each record carries its own TTL, chosen by the owner of the domain. Every teacher starts in the morning, while each recursive resolver starts counting at the moment it fetched the record. Recursive resolvers therefore switch at different times.

Your old A record had a TTL of 3600 seconds, one hour, and you changed the record at 10:00.

| Recursive resolver | Fetched old A record at | Returns old address until |
|---|---|---|
| Ana's recursive resolver | Held no copy | Returns the new address on its first query after 10:00 |
| Ben's recursive resolver | 09:50 | 10:50 |
| A third recursive resolver | 09:01 | 10:01 |

Ben's recursive resolver fetched the old record ten minutes before your change, so its copy had 50 minutes of TTL left at 10:00. Ana's recursive resolver had no copy, so its first query after 10:00 went to the authoritative server. No recursive resolver can hold a copy older than one TTL, so the last one switches by 11:00.

Each recursive resolver stores the old A record together with the old TTL. Lowering the TTL at 10:00 therefore changes nothing for copies fetched before 10:00. For a planned change, you lower the TTL first, wait one old TTL so that each cached copy carries the low value, and then change the address.

You can see both servers with dig, a command-line tool that sends one DNS query and prints the answer. I did not run these commands for this answer. The output lines show their form, and 192.0.2.10 is a documentation address.

1. Read the copy in your recursive resolver. Run `dig +noall +answer www.example.com A`, wait ten seconds, and run it again. Each run prints one line such as `www.example.com. 1742 IN A 192.0.2.10`. The second field is the remaining TTL in seconds. A number about ten lower on the second run means your recursive resolver answered from its cache.
2. Read the value at the source. An NS record is a record whose data is the name of an authoritative server for a domain. Run `dig +short NS example.com` to list those names. Then run `dig www.example.com A @ns1.example.net`, with one of the listed names after the `@`. The flags line of the output reads `flags: qr aa rd`. The `aa` flag, short for authoritative answer, is a bit in a DNS reply that marks a reply from an authoritative server. That reply carries the new address and the full TTL you configured.
3. Read the definition. RFC 1035, section 4.1.3, defines TTL as "a 32 bit unsigned integer that specifies the time interval (in seconds) that the resource record may be cached before it should be discarded." Resource record is the RFC's name for a record.

The one-TTL bound holds for recursive resolvers that respect the TTL. An operator can configure a recursive resolver to keep records longer, and that recursive resolver returns the old address past the TTL. Browsers and operating systems also keep short caches, so one client can lag behind its recursive resolver.

Your change took effect at the authoritative server at 10:00. Each recursive resolver picked it up when its own cached copy expired, at a time set by when it fetched the old record and capped by the old TTL. People call that spread of switch times DNS propagation, although authoritative servers push nothing to recursive resolvers.

## Why it fits

Glossary first, with exact definitions of every term the response uses. Big picture, then a concrete case before any mechanism. Analogy, then the exact version ending on the glossary wording, then the two places the analogy fails. A worked timeline with real times. Evidence the reader can repeat, with a note that the commands were not run. Limits, then a return to the whole.
