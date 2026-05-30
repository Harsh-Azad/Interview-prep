# Computer Networks: An Industry-Grade Study Primer for Satvik

**Audience.** Technical founder/engineer in Bengaluru preparing for Anthropic / OpenAI / DeepMind / FAANG / Databricks / Snowflake / Jane Street / Citadel / Two Sigma / Goldman / JPM / McKinsey Digital interviews.
**Bias.** Production systems, not OSI memorization. Cross-linked to OS, DistSys, Security, Databases, ML-infra.
**How to use.** Read tier-by-tier. For each topic, master the *concept summary* and *industry application*, then close the book and answer the *grill-down* prompts out loud. If you hesitate, go to the cited resource.

---

## How to read the tiers
- **Foundation (Weeks 1–3).** The wire. You will not pass a phone screen at any of these firms without fluency here.
- **Intermediate (Weeks 4–7).** The production stack. This is where Meta/Netflix/Google/Databricks system-design rounds live.
- **Advanced/Industry-Expert (Weeks 8–12).** The differentiator. Where Anthropic infra, DeepMind training, Jane Street kernel-bypass, and Cloudflare-grade questions live.

Anchor texts referenced throughout:
- Kurose & Ross, *Computer Networking: A Top-Down Approach* (KR)
- Stevens, *TCP/IP Illustrated Vol. 1* (Stevens)
- Grigorik, *High Performance Browser Networking* (HPBN, free at hpbn.co)
- Kleppmann, *Designing Data-Intensive Applications* (DDIA) — for CN ↔ DistSys/DB cross-links
- Google, *Site Reliability Engineering* and *The SRE Workbook* (free at sre.google)
- Stanford **CS144** (build a TCP/IP stack in C++ across labs 0–7, culminating in a TCP implementation that talks to real Internet servers), MIT **6.829**, CMU **15-441**

---

# TIER 1 — FOUNDATION

## 1.1 OSI vs TCP/IP and "why layering matters in industry"

**Concept.** OSI is a 7-layer pedagogical model; the Internet uses a 4-layer stack (Link → Internet → Transport → Application). What matters is the *contract at each layer* and where state lives.

**Industry application.** Layering decides *who owns what failure*: a Meta SRE debugging a 99.9-percentile spike must instantly classify it as L2 (NIC drops, PFC pause), L3 (BGP/ECMP rehash), L4 (TCP retransmits, TIME_WAIT), or L7 (HTTP/2 head-of-line block, gRPC deadline). Service meshes (Envoy/Istio) live at L7 but rely on L4 sockets; eBPF/Cilium *collapses* layers in the kernel for performance.

**Grill-down.**
- L1: Map TCP/IP → OSI. Which OSI layers are "fused" in TCP/IP?
- L2: Why does TLS sit "between" L4 and L7? What does that mean for a load balancer that wants to inspect HTTP headers?
- L3: A request is slow. Walk me from `getaddrinfo` → ARP/ND → IP → TCP → TLS → HTTP and identify *which layer's failure mode* produces each of: connection reset, slow start, certificate expired, 502, 504.
- L4: Why is QUIC said to "violate" layering? (Answer: it merges transport + crypto + part of session, runs in userspace over UDP.)

**Cross-links.** OS (sockets API exposes the L4/L7 boundary), Security (TLS as L5/6), SysDesign (the layer you cache at determines blast radius).

**Resources.** KR Ch. 1; HPBN Ch. 1–2.

---

## 1.2 Physical/Link layer essentials

**Concept.** Ethernet frame (preamble, MAC, EtherType, payload, FCS), MAC addresses (locally administered vs OUI-assigned), ARP (request who-has, reply is-at), switch vs hub (a hub is a 1-port collision domain; a switch maintains a CAM/MAC table and forwards selectively). VLANs (802.1Q tag), LACP/bonding, jumbo frames (MTU 9000 in DCs).

**Industry application.** In an AI training cluster, link-layer matters: jumbo frames cut per-packet overhead; PFC (Priority Flow Control, 802.1Qbb) is what makes RoCEv2 "lossless"; Meta's BE fabric uses non-blocking Ethernet leaf/spine because GPUs cannot tolerate drops on AllReduce.

**Grill-down.**
- L1: What's the MTU? Why 1500? What breaks if a hop has lower MTU? (Path MTU Discovery via ICMP "Frag Needed".)
- L2: Difference between a collision domain and a broadcast domain. What's the broadcast domain of a switch? A router?
- L3: How does ARP cache poisoning work and how do DCs defend against it? (Static ARP, dynamic ARP inspection, 802.1X.)
- L4 (HFT/AI): What is PFC and the head-of-line blocking it causes? Why does Meta tune PFC + DCQCN carefully on its RoCE BE network? (See §3.2.)

**Resources.** Stevens Ch. 4; *Ethernet: The Definitive Guide* (Spurgeon); Linux `ip`, `ethtool`.

---

## 1.3 IP, subnetting, CIDR, NAT, IPv4 vs IPv6

**Concept.** IPv4 is 32-bit; classful → CIDR (`/n` prefix). Subnet math by heart: `/24` = 256 addrs, `/16` = 65,536. RFC1918 private space: `10/8`, `172.16/12`, `192.168/16`. NAT (SNAT/DNAT, PAT) translates private↔public; breaks end-to-end principle. IPv6: 128-bit, no NAT by design, SLAAC, link-local `fe80::/10`.

**Industry application.** Every cloud VPC is just CIDR on top of overlay encapsulation (VXLAN/Geneve). AWS VPC: you pick a `/16`, carve subnets per AZ. NAT Gateway is a billed product because NAT is *stateful* and expensive. Kubernetes pod IPs typically use CNI plugins (Calico, Cilium) that allocate from a cluster CIDR.

**Grill-down.**
- L1: Convert `10.0.5.0/22` → range and host count.
- L2: Why can't two VPCs with overlapping CIDRs peer directly? How do you solve it? (Transit Gateway with NAT, or re-IP.)
- L3: Carrier-Grade NAT (CGNAT) — what breaks for mobile users? (Inbound connections, P2P, port-based rate limits.)
- L4: Why does IPv6 adoption matter for Anthropic/OpenAI serving Indian mobile users? (CGNAT exhaustion, Happy Eyeballs RFC 8305, DNS64/NAT64.)

**Cross-links.** Security (NAT as accidental firewall), Cloud (subnet = AZ in AWS), DistSys (split-brain in multi-region routing).

**Resources.** KR Ch. 4; AWS VPC docs.

---

## 1.4 Routing fundamentals (with industry-relevant BGP)

**Concept.** Static vs dynamic. Intra-domain: distance-vector (RIP, EIGRP) vs link-state (OSPF, IS-IS — each router has full topology and runs Dijkstra). Inter-domain: BGP, the *only* protocol that runs the public Internet. BGP is path-vector and policy-driven, not shortest-path.

**Industry application.** Cloudflare and Google anycast a single IP from hundreds of PoPs via BGP. AWS Direct Connect / Google Cloud Interconnect are essentially "give us your AS number and a BGP session." BGP misconfigs cause global outages: AS7007 in 1997 leaked 72,000+ routes; Facebook's October 2021 outage was a BGP withdrawal of its own DNS prefixes.

**Grill-down.**
- L1: What does BGP carry in an UPDATE? (NLRI, AS_PATH, NEXT_HOP, LOCAL_PREF, MED, communities.)
- L2: iBGP full-mesh vs route reflectors — why? (n² sessions; RRs scale to thousands of routers.)
- L3: Explain ECMP and why hash collisions matter for AI training. Per Meta SIGCOMM 2024: "ECMP rendered poor performance for the training workload due to the low flow entropy" — a few elephant flows hash to the same link.
- L4: Prefix hijacking, route leaks, and **RPKI ROAs**. Per the Cloudflare RPKI blog: RPKI lets IP holders cryptographically sign Route Origin Authorizations that bind a prefix to a permitted ASN with a max length. What does a ROA *not* protect against? (Path manipulation — that is BGPsec, barely deployed.)
- L5: Anycast: how does the same IP terminate in Singapore for a Bengaluru user and in Frankfurt for a Berlin user? Why does Cloudflare anycast 1.1.1.1 but Netflix Open Connect uses unicast + DNS steering?

**Cross-links.** SysDesign (anycast for global LB), Security (RPKI = PKI for routing).

**Resources.** KR Ch. 5; Cloudflare's RPKI blog series; *BGP* by Iljitsch van Beijnum; RFC 4271, RFC 4456.

---

## 1.5 TCP / UDP deep dive — *the* foundation interview topic

**Concept (TCP).**
- **3-way handshake.** SYN → SYN+ACK → ACK. ISN per side; SYN consumes a sequence number.
- **States.** CLOSED → LISTEN → SYN_SENT/SYN_RCVD → ESTABLISHED → FIN_WAIT_1 → FIN_WAIT_2 → TIME_WAIT (2·MSL) → CLOSED.
- **Reliability.** Cumulative ACK + SACK (RFC 2018), retransmit timer (RTO), fast retransmit (3 dup-ACKs), RACK (time-based loss detection).
- **Flow control.** Receiver advertises rwnd; sliding window. Window scaling (RFC 7323) for fat pipes.
- **Congestion control.** cwnd. Slow start (exponential) → congestion avoidance (additive) → fast recovery. Algorithms: **Reno** (loss-based), **CUBIC** (Linux default since 2.6.19, cubic function of time since last loss), **BBR** (model-based — see §3.x).

**Concept (UDP).** Stateless, no reliability, no ordering, no congestion control. Used for DNS, QUIC, RTP, video games, market-data multicast.

**Industry application — TIME_WAIT pain.** A connection-initiator that performs an active close stays in TIME_WAIT for 2·MSL. On Linux, this is hard-coded as `#define TCP_TIMEWAIT_LEN (60*HZ)` in `include/net/tcp.h` — about 60 seconds. With ephemeral port range 32768–60999 (~28k ports), a server initiating short-lived outbound connections caps at roughly 470 conn/sec per (src-ip, dst-ip, dst-port) tuple. Fixes (in order of preference): connection pooling, `SO_REUSEADDR` / `SO_REUSEPORT`, expand `ip_local_port_range`, multiple src IPs. Never blindly enable `tcp_tw_recycle` — it was removed in Linux 4.12 because it broke NAT.

**Industry application — BBR.** Per Cardwell et al. (ACM Queue 2016), BBR delivers 2–25× higher throughput than CUBIC on lossy paths. The paper states: *"BBR reduces median RTT by 53 percent on average globally, and by more than 80 percent in the developing world."*

**Grill-down — the canonical TCP onion.**
- **L1.** Walk me through the 3-way handshake. Why three, not two?
- **L2.** What happens if SYN-ACK is lost? (Initiator retransmits SYN with exponential backoff; Linux: `tcp_syn_retries=6` → ~127 s total.)
- **L3.** TCP states including TIME_WAIT, why 2·MSL, why this hurts at scale, and three different production fixes (pool, REUSEPORT, src-IP fan-out).
- **L4.** How does BBR differ from CUBIC, and when would you pick which on a CDN edge? (CUBIC: shallow buffers + clean networks. BBR: lossy mobile/transcontinental, deep buffers where you want to avoid bufferbloat. BBRv1 is unfair to CUBIC; BBRv2/v3 add loss + ECN response.)
- **L5.** TCP in a lossy 5G network vs datacenter — how does QUIC change the picture? (TCP conflates loss with congestion → cuts cwnd on random radio loss; QUIC stream-multiplexes so one lost packet does not HoL-block other streams; userspace congestion control allows per-app tuning.)
- **L6 (Anthropic/AI).** TCP between two GPU nodes 40 km apart — what is the maximum throughput at 10 ms RTT with default Linux buffers (~200 KB)? Compute BDP. This is why you do not use plain TCP for collective comms.

**Cross-links.** OS (`tcp_*` sysctls, socket buffer sizing, epoll), DistSys (timeouts as failure detectors), Security (SYN flood, SYN cookies).

**Resources.** Stevens Ch. 17–24; KR Ch. 3; Vincent Bernat, *Coping with TCP TIME-WAIT on busy Linux servers*; BBR paper (Cardwell et al., ACM Queue 2016); draft-ietf-ccwg-bbr; CS144 labs 1–4.

---

## 1.6 DNS deep dive

**Concept.** Hierarchy: root (13 IP addresses, hundreds of anycast instances per root-servers.org) → TLD → authoritative. Recursive resolver vs iterative. Caching at every layer (stub, recursor, OS, browser). Record types: A, AAAA, CNAME, ALIAS/ANAME, MX, TXT, SRV, CAA, HTTPS/SVCB (RFC 9460). TTL governs cache freshness. DoH (RFC 8484) / DoT (RFC 7858) for privacy. GeoDNS and EDNS Client Subnet (ECS) for steering.

**Industry application.** A DNS misconfiguration is the most common "global outage." Netflix Open Connect, Akamai, and Google all use **DNS-based steering** (close-to-user IP via ECS or anycast resolver) layered with anycast on the data path. **F-root** is anycast: per Cloudflare, "Since March 30, 2017, Cloudflare has been providing DNS Anycast service as additional F-Root instances under contract with ISC."

**Grill-down.**
- L1: A user types `claude.ai` — trace the resolution.
- L2: Difference between CNAME and ALIAS; why CNAME at apex is illegal (per RFC).
- L3: TTL trade-offs. You are cutting over an origin — do you lower TTL to 60s an hour before? What is the catch with stub resolvers that ignore TTL?
- L4: How does GeoDNS work with Cloudflare's anycast resolver chain? When does ECS leak privacy?
- L5: DoH vs DoT — why did Mozilla make DoH default? What is the resolver-centralization concern?

**Cross-links.** Security (DNSSEC, DoH for privacy, DNS cache poisoning), SysDesign (service discovery via DNS SRV records).

**Resources.** KR Ch. 2.4; *DNS and BIND* (Liu & Albitz); Cloudflare Learning Center.

---

## 1.7 HTTP/HTTPS — 1.1, 2, 3

**Concept.**
- **HTTP/1.1 (RFC 7230).** Text, headers + body, keep-alive default, pipelining theoretically allowed but broken in practice (HoL block). 6 parallel connections per origin in browsers.
- **HTTP/2 (RFC 9113).** Binary framing, **multiplexed streams** on a single TCP connection, HPACK header compression, server push (deprecated). One TCP HoL still applies — a single TCP loss stalls all streams on that connection.
- **HTTP/3 (RFC 9114).** Runs over QUIC over UDP. Streams are independent; per-stream HoL only. QPACK replaces HPACK. 0-RTT resumption. Connection migration across IPs (Wi-Fi → 5G).

**Industry data.** Per Cloudflare's *Radar 2025 Year in Review*: *"Globally in 2025, 50% of requests to Cloudflare were made over HTTP/2, HTTP/1.x accounted for 29%, and the remaining 21% were made via HTTP/3. These shares are largely unchanged from 2024."* Inside datacenters, HTTP/3 is usually overkill — RTTs are sub-millisecond, multiplexing wins are marginal; HTTP/2 over a long-lived connection dominates.

**Grill-down.**
- L1: Why is HTTP stateless and how do cookies repair that?
- L2: HTTP/1.1 → 2 — what concretely improves and what does not? (Latency on parallel resources improves; throughput on a single big object does not.)
- L3: HoL block — explain it at three layers: HTTP/1.1 (head request blocks pipeline), HTTP/2 (TCP segment loss blocks all streams), HTTP/3/QUIC (only the affected stream).
- L4: When would you NOT enable HTTP/3 at your edge? (UDP-blocked enterprise firewalls, origin lacks support, CPU saturation on userspace crypto.)

**Resources.** HPBN Ch. 11–12; RFC 9114; Cloudflare blog on HTTP/3.

---

## 1.8 TLS handshake (1.2 vs 1.3), mTLS, ciphers

**Concept.**
- **TLS 1.2:** ClientHello → ServerHello + cert + KeyExchange → client key exchange → Finished. 2 RTTs.
- **TLS 1.3 (RFC 8446):** 1-RTT default; **0-RTT** with PSK resumption (replay-vulnerable for non-idempotent requests). Removed RSA key exchange, CBC, SHA-1, compression. AEAD-only (AES-GCM, ChaCha20-Poly1305).
- **mTLS:** both sides present certs. The default for service-mesh east-west traffic (Istio, Linkerd, Cilium auto-rotate via SPIFFE/SVID).
- **Cert chain:** leaf → intermediate(s) → root in OS/browser store. **OCSP stapling** for revocation.

**Industry application.** Anthropic/OpenAI inference APIs terminate TLS 1.3 at the edge LB and re-encrypt upstream with mTLS in the service mesh. 0-RTT is great for static page reloads (Cloudflare ships it via `ssl_early_data`) but disabled by default for POST. Per the Cloudflare 0-RTT blog: 0-RTT lets a client send application data before the TLS handshake completes by reusing keys from a prior session, but origins that handle non-idempotent endpoints should answer with HTTP 425 (Too Early).

**Grill-down.**
- L1: Why do we need both symmetric and asymmetric crypto in TLS?
- L2: TLS 1.3 — what is removed vs 1.2 and why? (Forward secrecy mandatory, no RSA key exchange.)
- L3: 0-RTT replay risk — concretely, name a request you would never send in 0-RTT.
- L4: mTLS at scale — how does Istio rotate certs every 24 h without downtime? (SDS, per-pod cert via SPIFFE ID.)
- L5: Forward secrecy — what does an attacker who later steals the server's long-term key learn about past sessions?

**Cross-links.** Security (PKI, SPIFFE), DistSys (cert rotation as a distributed problem).

**Resources.** RFC 8446; *Bulletproof TLS and PKI* (Ristić); Cloudflare blog "Even faster connection establishment with QUIC 0-RTT resumption."

---

# TIER 2 — INTERMEDIATE (Production Stack)

## 2.1 Load balancing — L4 vs L7, algorithms, Maglev

**Concept.**
- **L4 LB** (HAProxy in TCP mode, AWS NLB, Google Maglev): operates on 5-tuple, no payload inspection, line-rate, DSR (Direct Server Return) capable.
- **L7 LB** (Envoy, Nginx, AWS ALB): parses HTTP, can route by header/path/cookie, can terminate TLS.
- **Algorithms.** Round-robin, weighted RR, least-connections, least-pending-requests (P2C — Twitter Finagle, Envoy default), **EWMA / least-latency**, **consistent hashing** (Karger; ring hash), and **Maglev hashing** (Google, NSDI 2016 — fixed-size lookup table, faster than ring, slightly worse on backend churn).

**Industry application.** Per Eisenbud et al., NSDI 2016: "Network routers distribute packets evenly to the Maglev machines via Equal Cost Multipath (ECMP); each Maglev machine then matches the packets to their corresponding services and spreads them evenly to the service endpoints… a single Maglev machine is able to saturate a 10 Gbps link with small packets." Envoy and Cloudflare Unimog use Maglev hashing for session affinity that survives backend churn.

**Grill-down.**
- L1: When pick L4 vs L7?
- L2: Round-robin's failure mode? (Stateless backends, no instance-state awareness — slow node gets equal share, latency spikes.)
- L3: Consistent hashing ring vs Maglev — what does Maglev optimize for? (Even load distribution, fast O(1) lookup, accepts slightly higher disruption on backend change.)
- L4: How does an L4 LB do DSR and why does it matter for video CDN egress? (Replies bypass LB → 10× throughput.)
- L5: Two LBs in front of three backends, one backend dies — what does consistent hashing buy you vs RR? (Only 1/N of sessions rehash, not all.)

**Cross-links.** DistSys (consistent hashing originated in DHTs — Chord/Dynamo), SysDesign (load balancing is *the* canonical sysdesign primitive).

**Resources.** Eisenbud et al., *Maglev: A Fast and Reliable Software Network Load Balancer* (NSDI 2016); Envoy `lb_policy` docs; *SRE Workbook* "Managing Load."

---

## 2.2 CDN architecture (Netflix Open Connect, Cloudflare)

**Concept.** Edge PoPs cache static + cacheable dynamic content close to users. **Anycast** routes to nearest PoP. **Cache hierarchy:** edge → regional/parent ("origin shield") → origin. **Cache key** = method + URL + Vary headers. **Invalidation:** TTL, purge API, surrogate keys (Fastly), stale-while-revalidate. **Push vs pull CDNs.**

**Industry application.** Netflix places **Open Connect Appliances (OCAs)** inside ISP networks, pre-positioning popular content during off-peak fill windows. Per Netflix Open Connect (as quoted in APNIC Blog, June 2018): *"Close to 95% of our traffic globally is delivered via direct connections between Open Connect and the residential ISPs our members use to access the Internet."* Cloudflare uses pure anycast (a single IP handled by hundreds of PoPs). Netflix's Live Origin (2025 blog) sets `Cache-Control: max-age=5` on 503s during overload to dampen retry storms.

**Grill-down.**
- L1: What is cacheable in HTTP? (Idempotent methods, status 200/203/300/301/404/410 by default, respect `Cache-Control`, `Vary`.)
- L2: Cache stampede / thundering herd at TTL expiry — mitigations? (`stale-while-revalidate`, request coalescing on the edge, jittered TTLs.)
- L3: Anycast CDN vs DNS-steered CDN: trade-offs? (Anycast: simpler, fast failover; DNS: finer-grained geo, but ECS leaks privacy and TTL caching makes failover slow.)
- L4: How does Netflix decide which content to fill into which OCA? (Predictive popularity + steering tables; off-peak fills.)
- L5: Origin shield — why add an extra hop? (Reduces origin RPS; deduplicates parallel misses across edge PoPs.)

**Cross-links.** SysDesign (caching is recurring), DistSys (cache invalidation is "one of the two hard problems").

**Resources.** Netflix Open Connect public docs (openconnect.netflix.com); Cloudflare blog `tag/cdn`; *Web Caching* (Wessels).

---

## 2.3 Reverse / forward proxies — Nginx, Envoy, HAProxy

**Concept.** Reverse proxy sits in front of servers (for TLS termination, caching, LB, WAF). Forward proxy sits in front of clients (egress control, corporate filtering). Architecture choices: process-per-core (Envoy, Nginx with `worker_processes auto`), event loops, shared listening sockets (`SO_REUSEPORT`).

**Industry application.** Envoy (originated at Lyft, now CNCF) is the data plane of Istio, AWS App Mesh, GCP Cloud Service Mesh, and Cloudflare's recent stack. Its xDS API (LDS/RDS/CDS/EDS/SDS) is now the de facto standard for dynamic proxy config.

**Grill-down.**
- L1: Nginx vs Envoy — design philosophy? (Nginx: static config + master/worker C; Envoy: dynamic xDS + filter chains in C++.)
- L2: How does an HTTP/2 proxy avoid amplifying the HoL problem? (Per-stream backpressure via WINDOW_UPDATE; but a slow upstream still stalls a single stream.)
- L3: Where does TLS terminate in your architecture and why? (At the edge for performance; re-encrypt to upstream with mTLS for zero-trust.)
- L4: Envoy filter chain — name a filter you would write yourself and at which layer it sits.

**Resources.** Envoy docs; Matt Klein's "Service Mesh Data Plane vs Control Plane."

---

## 2.4 Service discovery — DNS, Consul, etcd

**Concept.** Static config → DNS round-robin → DNS SRV → dedicated registry (Consul, ZooKeeper, etcd, Eureka). Kubernetes Service is a DNS-based registry built on kube-proxy iptables/IPVS/eBPF.

**Grill-down.**
- L1: Why is DNS RR a bad load balancer for microservices? (TTL caching, no health check.)
- L2: Eventual vs strong consistency in service discovery — Consul (Raft, strong by default with stale-reads opt-in) vs Eureka (gossip, eventual).
- L3: Headless services in Kubernetes — when use them? (StatefulSets like Kafka where pods need stable identity.)

**Cross-links.** DistSys (Raft, gossip), DB (service registry as a tiny DB).

---

## 2.5 API gateways & rate limiting

**Concept.** Gateway = AuthN/Z + routing + rate-limit + quota + transformation. Rate-limit algorithms:
- **Token bucket:** tokens drip in at rate r, capacity b, request consumes 1; allows bursts up to b.
- **Leaky bucket:** queue with constant drain; smooths bursts.
- **Fixed window:** count per minute. Easy, but 2× burst at window boundary.
- **Sliding window log / counter:** weighted blend of prior + current window — Cloudflare uses this; ~5× cheaper than a full log.

**Industry application.** OpenAI's API uses token-bucket-like quotas per org (TPM/RPM). Cloudflare's distributed rate limit uses a coordinated counter (eventually consistent across PoPs).

**Grill-down.**
- L1: Token vs leaky bucket — which gives better burst tolerance?
- L2: Distributed rate limiting across 100 LBs — strict (Redis Lua, INCR with TTL) vs approximate (local counter + periodic gossip)?
- L3: How would you rate-limit by *tokens consumed*, not requests? (LLM-specific — bill on output tokens, decrement after completion, allow temporary debt.)

**Resources.** Stripe blog "Scaling your API with rate limiters"; Cloudflare blog on sliding window.

---

## 2.6 WebSockets, SSE, long polling — when to use what

| Need | Use |
|---|---|
| Bidirectional, low latency, full-duplex | WebSocket |
| Server-push only, simple, HTTP-friendly, auto-reconnect | **Server-Sent Events (SSE)** |
| Legacy / no WS support / cannot run UDP | Long polling |
| Sub-millisecond, ordered, finance | gRPC bidi or raw TCP |

**Industry application.** LLM chat streaming uses **SSE** almost universally (ChatGPT, Claude, Gemini) — it is HTTP, plays well with CDNs/L7 LBs, supports `Last-Event-ID` for resume, and avoids WebSocket framing issues with reverse proxies.

**Grill-down.**
- L1: WS handshake — what is special? (HTTP upgrade with `Connection: Upgrade, Sec-WebSocket-Key`.)
- L2: WS through an L7 LB that does not support upgrade — what happens?
- L3: SSE vs WS for an LLM chat — why SSE wins. (Unidirectional, simpler, HTTP/2 multiplexing helps, no need for ping/pong.)
- L4: SSE over HTTP/2 with 100 streams per connection — what is the bottleneck? (Per-connection `SETTINGS_MAX_CONCURRENT_STREAMS`, typically 100; multiplexing on slow clients adds backpressure.)

---

## 2.7 gRPC vs REST vs GraphQL — networking implications

**Concept.** REST: JSON over HTTP/1.1 or 2, human-readable, cacheable. gRPC: protobuf binary over HTTP/2 (mandated), four streaming modes (unary, server-stream, client-stream, bidi). GraphQL: single endpoint, client-specified query shape, hard to cache at edge.

**Industry application.** Per Microsoft's gRPC best-practices: HTTP/2 limits concurrent streams (typically 100/connection); a single gRPC channel can become the bottleneck — solution: channel pool or `SocketsHttpHandler.EnableMultipleHttp2Connections=true`. L4 LBs do not balance gRPC well (single TCP, all streams pinned). Use client-side LB (xDS, gRPC-LB) or L7 proxy (Envoy).

**Grill-down.**
- L1: Why does gRPC mandate HTTP/2? (Streams, trailers for status codes, binary framing.)
- L2: gRPC over an L4 LB — what is the failure mode? (One backend gets all streams of one client; uneven load.)
- L3: GraphQL caching at the edge — why hard? (POST + query in body; mitigations: persisted queries, GET with query hash.)
- L4: gRPC-web — why does it exist, what is missing? (Browsers cannot speak HTTP/2 trailers reliably; no client-streaming; no bidi.)

**Resources.** grpc.io performance docs; Arpit Bhayani "Why gRPC uses HTTP/2."

---

## 2.8 Network security — firewalls, WAF, DDoS, zero trust

**Concept.** Stateful firewall (conntrack) vs stateless ACL. WAF: L7 inspection for OWASP top-10 (SQLi, XSS, deserialization). DDoS layers: volumetric (Gbps, mitigated upstream at scrubbing centers), protocol (SYN flood — SYN cookies; UDP amplification — DNS/NTP/Memcached reflection), application (slow-loris, HTTP flood — bot management, JS challenges). **Zero Trust** (BeyondCorp at Google): no implicit perimeter trust; every request authenticated + authorized; identity-aware proxy.

**Grill-down.**
- L1: SYN flood — what is it and how do SYN cookies defend?
- L2: A UDP amplification attack (DNS) reaches 1 Tbps. What is your layered defense? (Anycast scrubbing, rate-limit per src, BCP38 ingress filtering upstream.)
- L3: Zero trust vs perimeter — what concretely changes? (No "inside" trust, mTLS everywhere, per-request authZ via OPA/Cedar.)
- L4: HTTP/2 Rapid Reset (CVE-2023-44487) — what was the attack, how was it mitigated?

**Resources.** Cloudflare Learning Center DDoS; BeyondCorp papers (Google research); OWASP.

---

## 2.9 Cloud VPC, subnets, security groups, NACLs, VPN

**Concept (AWS-flavored; others parallel).**
- **VPC:** /16 CIDR, AZ-scoped subnets, route tables.
- **Security group:** stateful, instance-level, default-deny inbound.
- **NACL:** stateless, subnet-level, ordered rules.
- **VPC peering:** non-transitive; **Transit Gateway** for hub-and-spoke.
- **VPN / Direct Connect:** IPsec over Internet vs dedicated fiber + BGP.
- **PrivateLink / Private Service Connect:** expose a service across VPC boundary via interface endpoints — avoid public Internet.

**Industry application.** Per Snowflake docs: "AWS PrivateLink is an AWS service for creating private VPC endpoints that allow direct, secure connectivity between your AWS VPCs and the Snowflake VPC without traversing the public internet." Databricks docs describe two endpoints: a **workspace endpoint** for REST API + classic compute plane traffic, and an **SCC relay endpoint** for secure cluster connectivity between compute plane and control plane.

**Grill-down.**
- L1: SG vs NACL — when use which?
- L2: VPC peering vs Transit Gateway — when does TGW win? (>2 VPCs, transitive routing, central egress.)
- L3: Why PrivateLink and not VPC peering for SaaS like Snowflake/Databricks? (PrivateLink is unidirectional, exposes one service IP, no CIDR-overlap concerns, multi-tenant.)

**Resources.** AWS *Well-Architected — Networking* pillar; AWS Networking Workshops.

---

## 2.10 Service mesh — Istio, Linkerd, Cilium

**Concept.** Sidecar pattern (Envoy injected next to each pod) → data plane. Control plane (Istiod) pushes config via xDS. Mesh provides: mTLS, traffic shifting (canary/A-B), retries, circuit breaking, distributed tracing headers, RBAC.

**Industry application.** Per Cilium docs: "Cilium Service Mesh delivers the core capabilities of a traditional service mesh — L7 traffic management, mutual TLS, and rich observability — using eBPF to avoid the overhead of per-pod sidecars." Istio Ambient Mode removes sidecars in favor of a per-node `ztunnel` plus optional waypoint proxies, saving substantial CPU. Anthropic and most modern infra teams choose Istio or Cilium depending on whether they want full L7 features or eBPF performance.

**Grill-down.**
- L1: Why a sidecar instead of a library? (Polyglot, transparent, decouples ops from app.)
- L2: Sidecar cost — CPU/memory per pod, double TCP hops. When is it not worth it?
- L3: Ambient mesh — what changes architecturally? (L4 in `ztunnel` DaemonSet; L7 only when needed via waypoint.)
- L4: How does mTLS auto-rotation work? (SPIFFE SVID, SDS pushes new cert, hot-reload at proxy.)

**Resources.** istio.io docs; *Istio in Action* (Posta); Cilium docs.

---

## 2.11 Observability — distributed tracing, OpenTelemetry, network telemetry

**Concept.** Three pillars: metrics (Prometheus, time-series), logs (structured JSON, Loki/ELK), traces (Jaeger/Tempo/Honeycomb). **W3C Trace Context** (`traceparent` header) propagates trace+span IDs. **OpenTelemetry** is the vendor-neutral SDK + OTLP wire protocol. Network telemetry: sFlow, IPFIX, eBPF-based (Pixie, Cilium Hubble).

**Grill-down.**
- L1: Why is trace context propagation hard across async boundaries? (Queues, threads; need explicit context handoff.)
- L2: Sampling strategies — head-based vs tail-based; why tail-based needs a collector buffering decisions.
- L3: P99 vs P99.9 — why does a service with great P99 still have terrible P999? (See §3.11 tail latency.)

**Resources.** Dapper paper (Google 2010); OpenTelemetry spec.

---

# TIER 3 — ADVANCED / INDUSTRY-EXPERT

## 3.1 Kernel bypass — DPDK, eBPF/XDP, AF_XDP, io_uring

**Concept.**
- **DPDK:** PMD (Poll Mode Drivers) in userspace; NIC mapped via UIO/VFIO; no interrupts, no kernel; busy-poll. Sub-microsecond per-packet processing; saturates 100/200 Gbps with the right NIC.
- **XDP (eXpress Data Path):** eBPF program at the driver hook, *before* sk_buff allocation. Can DROP, PASS, TX, REDIRECT. Used by Cloudflare DDoS, Cilium, Facebook Katran load balancer.
- **AF_XDP:** zero-copy userspace socket family, "DPDK-like" but lives inside the Linux ecosystem.
- **io_uring:** async I/O ring buffers between userspace and kernel; one syscall amortized over many ops; supports network, disk, registered buffers, fixed files.

**Industry application.** Industry write-ups on kernel bypass HFT report that traditional Linux networking adds roughly 20–50 µs/transit and DPDK/RDMA cut kernel-contributed latency to roughly 1–5 µs. Cloudflare migrated from iptables+xt_bpf to XDP-based DDoS mitigation; per Cilium docs, this "combined best of both worlds by having high-performance programmable packet processing directly inside the kernel." Cilium replaces kube-proxy with eBPF-based service load balancing.

**Grill-down.**
- L1: Why is the kernel slow for packet I/O? (Syscall + context switch + skb alloc + interrupts + copies.)
- L2: DPDK vs XDP — when use which? (DPDK: dedicated NIC, max perf, you own the stack. XDP: keep the kernel, surgical hooks, dynamic via eBPF.)
- L3: AF_XDP — what does it give you over raw AF_PACKET? (Zero-copy via UMEM, comparable to DPDK perf but with kernel semantics.)
- L4: io_uring vs epoll — concrete latency and throughput difference for a 1M-rps HTTP server.
- L5: Cilium uses eBPF host-routing to "bypass iptables and the upper host stack" — what does that buy in microseconds?

**Cross-links.** OS (NAPI, RPS/RFS, GRO/GSO, NUMA-pinning), HFT, Cloud-native networking (Cilium).

**Resources.** dpdk.org; Cilium BPF reference guide; Brendan Gregg's eBPF book; *Linux Kernel Networking* (Rosen).

---

## 3.2 RDMA, RoCE, InfiniBand — AI training fabric

**Concept.**
- **RDMA:** NIC writes/reads directly into remote VA without CPU involvement. Verbs API: QP (Queue Pair), CQ, MR (Memory Region). Transport types: RC (Reliable Connected, like TCP), UC, UD.
- **InfiniBand (IB):** dedicated lossless fabric (Mellanox/NVIDIA), per-link credit-based flow control, hop latency ~100 ns.
- **RoCEv2:** RDMA over UDP/IP/Ethernet — needs PFC (Priority Flow Control) + ECN-based DCQCN to be lossless-enough.
- **iWARP:** RDMA over TCP — easier on the network, worse perf.

**Industry application.** Per Gangidi et al., *RDMA over Ethernet for Distributed AI Training at Meta Scale* (SIGCOMM 2024): Meta separates FE and BE networks; the BE is "a specialized fabric that connects all RDMA NICs in a non-blocking architecture, providing high bandwidth, low latency, and lossless transport between any two GPUs in the cluster." Meta moved off RoCEv1 star to a RoCEv2 fabric; ECMP failed with low flow entropy in AI workloads; DCQCN settings tuned aggressively were rolled back because PFC pressure worsened 2–3× for marginal completion-time gains. xAI's Colossus targets 100k+ GPUs in a single RDMA fabric at 400 Gbps ports.

**Grill-down.**
- L1: Why does RDMA bypass the CPU? (NIC has its own MMU; registered MR pinned in DRAM.)
- L2: PFC — what is it, what is a "PFC storm"? (Pause frames per priority class; if not configured well, deadlocks propagate back across the fabric.)
- L3: DCQCN — what signal does it use? (ECN marks on switches → NIC reduces rate.)
- L4: Why does ECMP fail for AI training? (5-tuple hash with few flows → polarization; per Meta, "ECMP rendered poor performance for the training workload due to the low flow entropy.")
- L5: RoCEv2 vs InfiniBand — why does Meta pick Ethernet despite IB being better in raw performance? (Vendor flexibility, operational tooling, switch ecosystem.)

**Resources.** Meta SIGCOMM 2024 paper; NVIDIA InfiniBand handbook; *Designing High-Performance Distributed Systems* (Hoefler).

---

## 3.3 GPU networking — NVLink, NVSwitch, GPUDirect RDMA, H100/B200 wiring

**Concept.**
- **NVLink 4 (Hopper):** 18 links × 25 GB/s/direction → 900 GB/s bidirectional per H100.
- **NVSwitch (3rd gen, in DGX H100):** 4 NVSwitch ASICs per server, full any-to-any inside an 8-GPU node, total all-to-all bisection of 7.2 TB/s.
- **NVL72 (Blackwell era):** extends NVLink switching to 72 GPUs across multiple trays as one "GPU domain"; NVLink Switch System scales up to 256 H100 GPUs with 57.6 TB/s all-to-all bisection per NVIDIA's third-gen NVSwitch documentation.
- **GPUDirect RDMA:** NIC DMA directly into GPU HBM (no CPU/host-DRAM bounce).
- **Topology layering:** TP (tensor parallel) inside NVLink domain (highest BW), DP/PP cross-node over RoCE/IB.

**Industry application.** A modern H100 SuperPOD: 8 H100/server, 4 NVSwitch, 8× ConnectX-7 (400 Gbps each) for scale-out, rail-optimized two-tier Clos. NVL72 fits TP-72 (or TP-8 × DP-9) entirely on NVLink, dropping cross-node traffic by orders of magnitude.

**Grill-down.**
- L1: NVLink vs PCIe — bandwidth and latency?
- L2: Why is TP confined to a node usually? (Activation transfer volume; outside NVLink domain it becomes the bottleneck.)
- L3: GPUDirect RDMA — what does it remove from the critical path? (Host-DRAM bounce, CPU sync.)
- L4: Wire a 1024-GPU H100 cluster optimally for a 70B-param dense LLM with TP=8, PP=4, DP=32. Where does each parallelism axis live in the network?
- L5: NVL72 changes what about training topology? (TP can be 72; a whole DGX-class node now spans a rack.)

**Cross-links.** ML infra (training parallelism); hardware (HBM bandwidth determines decode perf).

**Resources.** NVIDIA H100 / GB200 whitepapers; NVIDIA Technical Blog "Upgrading Multi-GPU Interconnectivity with the Third-Generation NVSwitch"; "AI Data Center Networking" (thenetworkdna).

---

## 3.4 Collective communication — AllReduce, AllGather, ReduceScatter, NCCL internals

**Concept.**
- **Primitives:** AllReduce (sum + bcast), AllGather, ReduceScatter, Broadcast, AlltoAll.
- **Algorithms:** **Ring AllReduce** (2(k−1) steps; bandwidth-optimal for large messages). **Tree AllReduce** (log k steps; latency-optimal for small messages). **Double-binary tree** (NCCL default for inter-node small messages). **NVLS** (NVLink SHARP — in-switch reduction on NVSwitch 3+).
- **Topology awareness:** NCCL auto-detects PCIe/NVLink/NVSwitch/RoCE/IB and builds rings/trees. **PXN** (NCCL 2.12+) aggregates messages through NVLink to reduce spine-switch traffic.
- **AllToAll:** O(N²) messages, dominates MoE training; cannot use ring/tree → needs careful scheduling.

**Industry application.** Per *Demystifying NCCL* (arXiv 2507.04786): "The choice of algorithm, typically ring or tree, depends on the specific collective operation and relevant execution parameters such as message size and topology." Production training tunes `NCCL_ALGO`, `NCCL_PROTO`, `NCCL_NCHANNELS`. MoE models (Mixtral, GPT-4-class) are AllToAll-dominated, motivating rail-optimized topologies and SHARP.

**Grill-down.**
- L1: AllReduce in 1 sentence + bandwidth lower bound. (Per GPU sends 2(N−1)/N · data.)
- L2: Ring vs tree — when does each win? (Ring: large messages; tree: small.)
- L3: Map AllReduce to a fat-tree topology. Why does ring "wrap around" badly across a slow link?
- L4: AllToAll in MoE — why is it the new bottleneck and how do rail-optimized networks help? (Each rank talks to all; pinning ranks to NIC "rails" gives single-hop reachability.)
- L5: NCCL_NVLS — what does in-switch reduction (SHARP) do for AllReduce latency at small messages?

**Cross-links.** ML infra; DistSys (gossip vs structured collectives).

**Resources.** NCCL docs; *Demystifying NCCL* (arXiv 2507.04786); TACCL (arXiv 2111.04867); NVIDIA blog "Doubling all2all with NCCL 2.12."

---

## 3.5 AI cluster topology — fat-tree, rail-optimized, dragonfly+, Jupiter/Aquila

**Concept.**
- **Fat-tree / Clos:** non-blocking multistage; full bisection bandwidth if oversubscription is 1:1.
- **Rail-optimized:** each GPU's i-th NIC connects to the i-th leaf switch ("rail"); same-rank GPUs across servers share a rail and reach each other in one hop.
- **Dragonfly+:** groups of switches fully meshed locally + globally — HPC (Slingshot at HPE Cray).
- **Direct-connect (Google Jupiter Evolving, SIGCOMM 2022):** removes spines; aggregation blocks connect directly via **MEMS-based Optical Circuit Switches (OCS)** dynamically reconfigured. Per Poutievski et al.: *"In this period Jupiter has delivered 5x higher speed and capacity, 30% reduction in capex, 41% reduction in power, incremental deployment and technology refresh all while serving live production traffic."* Per Google Cloud Blog ("Speed, scale and reliability: 25 years of Google datacenter networking evolution"): *"We support 64 such blocks for a total bisection bandwidth of 64 × 204.8 Tb/s = 13.1 Pb/s … our current fifth-generation Jupiter data center network architecture, which now scales to 13 Petabits/sec of bisectional bandwidth."*
- **Meta RoCE BE:** two-tier Clos AI Zone; RTSW leaves connect servers via copper DAC; CTSW spines provide scale-out.

**Grill-down.**
- L1: Bisection bandwidth — define and compute it for a 3-layer fat-tree.
- L2: Why rail-optimization? Why does it pair so well with ring AllReduce?
- L3: Optical circuit switching — why does Google use it inside a DC? (Reconfigure topology to traffic; bypass spine bottleneck.)
- L4: Compare Meta's RoCE BE design to Google's Jupiter direct-connect — what are they optimizing for?

**Resources.** SIGCOMM 2022 *Jupiter Evolving* (Poutievski et al.); SIGCOMM 2024 *RDMA over Ethernet at Meta Scale*; *Aquila* (NSDI 2022); Google Cloud Blog "Jupiter now scales to 13 Petabits per second."

---

## 3.6 BGP deep dive — RR, communities, hijack, cloud peering

**Concept.** iBGP route reflectors (RFC 4456) — break the n² requirement; clusters of RRs for redundancy. BGP communities: tags (e.g., `64500:100`) influence policy (no-export, prepend, geo-tag). **AS_PATH prepending** to influence inbound traffic. **MED, LOCAL_PREF** for outbound. **Cloud BGP:** AWS Direct Connect, GCP Interconnect, Azure ExpressRoute all use BGP — you announce your prefixes, they announce cloud prefixes; private peering avoids public Internet. **RPKI ROV** drops invalid origins.

**Grill-down.**
- L1: What is a community? Give two uses.
- L2: You have two ISPs. Make traffic prefer ISP-A inbound. (Prepend on ISP-B announcement, or get ISP-A to announce a more specific prefix.)
- L3: RPKI catches origin hijack but not path hijack — explain. What is BGPsec and why is it un-deployed?
- L4: AWS Direct Connect BGP gotchas: BFD timers, max prefixes, jumbo frames.

**Resources.** RFC 4271, RFC 4456, Cloudflare RPKI blog series, NLNOG ring docs.

---

## 3.7 Anycast at scale

**Concept.** A single IP announced from many locations via BGP; routers pick "nearest" by AS_PATH and IGP cost. Used by **all 13 DNS root server IPs** (root-servers.org lists 1700+ instances), Cloudflare 1.1.1.1, Google 8.8.8.8, Cloudflare Workers, Fastly. Per the Cloudflare F-root announcement, Cloudflare has provided additional F-root anycast instances under contract with ISC since March 30, 2017.

**Grill-down.**
- L1: Anycast for UDP (DNS) is easy; anycast for TCP — what breaks? (Mid-connection route shift can land you on a different PoP. Solutions: stable hashing at PoP ingress, session affinity via cookie.)
- L2: Anycast vs DNS geo-steering — pros and cons. (Anycast: instant failover via BGP withdraw; DNS: finer per-user steering but TTL-bound.)
- L3: Cloudflare's "Unimog" L4 LB uses Maglev hashing on top of anycast — why?
- L4: A BGP misconfig at one PoP starts black-holing anycast traffic — how do you detect + drain?

---

## 3.8 QUIC internals

**Concept.** Per RFC 9000: UDP-based, streams (independent reliable byte streams within a connection), CIDs (connection IDs decouple connection from 5-tuple — enables migration), built-in TLS 1.3 handshake, packet-number encryption, 0-RTT via PSK. Pacing emitted via `SendInfo.at` (Cloudflare quiche); kernel `SO_TXTIME` for hardware-paced sends.

Per Cloudflare: "QUIC separates out the layer 4 transport connection from the layer 3 IP flow, allowing for migration between different networks without disruption."

**Grill-down.**
- L1: Why UDP? (Faster iteration than baking new TCP options into middleboxes; encryption pierces middlebox ossification.)
- L2: HoL block: TCP vs QUIC under 1% packet loss with 10 parallel HTTP requests.
- L3: Connection migration — how does the server know the new path is legitimate? (Path validation via `PATH_CHALLENGE`/`PATH_RESPONSE`.)
- L4: 0-RTT replay — what does the application have to do? (Distinguish safe/idempotent ops; HTTP 425 Too Early.)
- L5: QUIC userspace cost — why does it burn more CPU than TCP+TLS? (No GSO/GRO offload historically; per-packet crypto.)

**Resources.** RFC 9000–9002; cloudflare-quic.com; Daniel Stenberg's HTTP/3 explainer.

---

## 3.9 HTTP/3 in production

**Concept.** Same semantics as HTTP/2 mapped to QUIC streams. QPACK (RFC 9204) replaces HPACK with stream-decoupled tables. Alt-Svc / SVCB-HTTPS DNS record advertise H3 endpoints.

**Industry data.** Per Cloudflare *Radar 2025 Year in Review*: HTTP/2 still serves 50% of Cloudflare requests, HTTP/1.x 29%, and HTTP/3 21% — *"largely unchanged from 2024."* Adoption blockers: UDP-blocked enterprise firewalls, no origin H3 in many stacks (Cloudflare→origin still H2/H1), server CPU.

**Grill-down.** See §1.7 and §3.8 combined.

---

## 3.10 Multi-region & multi-cloud networking

**Concept.** Cross-region replication (S3 CRR, Spanner global), transit gateways (AWS TGW, GCP NCC), Global Accelerator (anycast IP that lands users on nearest AWS region). Inter-region latency budgets: us-east-1 ↔ eu-west-1 ~ 80 ms, us-east-1 ↔ ap-south-1 ~ 200 ms.

**Grill-down.**
- L1: Why is multi-region active-active so much harder than active-passive?
- L2: Cross-region consistency — eventual vs causal vs strong. What is the latency cost of strong across regions? (≥1 cross-region RTT per quorum write.)
- L3: Egress costs — why are they the silent killer of multi-cloud?

**Cross-links.** DDIA Ch. 5, 9.

---

## 3.11 Tail latency — P50/P99/P999, HoL, bufferbloat, GC

**Concept.** A request fan-out to N backends experiences max-of-N latency → P99 of one backend dominates user P50 for N≈100. Causes: **HoL blocking** (TCP loss, HTTP/1.1 pipelining, single-queue NIC), **GC pauses** (JVM, Go ≤1.4), **CPU throttling** (cgroup CFS), **network queuing** (bufferbloat in deep middlebox buffers — BBR's raison d'être).

**Industry application.** Dean & Barroso 2013 *The Tail at Scale* — hedged requests, tied requests, micro-partitioning, request priority. Netflix Live Origin returns 503 with `max-age=5` to dampen retry storms.

**Grill-down.**
- L1: Why is P999 worse than 10× P99 in fan-out systems?
- L2: Bufferbloat — what is it, how does CoDel/FQ-CoDel + BBR address it?
- L3: Two GC-bound services in a chain — what is the latency math? (Convolutions of pause distributions.)
- L4: How would you measure tail latency for an LLM SSE stream — TTFT vs ITL vs total?

**Resources.** Dean & Barroso *The Tail at Scale* (CACM 2013); *SRE Book* Ch. 21–22.

---

## 3.12 TCP optimization — Nagle, delayed ACK, window scaling, SACK, ECN, TFO, BBRv2/v3

**Linux sysctls and socket options to know cold.**
- `TCP_NODELAY` disables Nagle (combine small writes); essential for interactive RPC.
- `TCP_QUICKACK` defeats delayed-ACK (200 ms by default; kernel auto-reverts after one quick ACK, so re-set every read).
- `net.ipv4.tcp_window_scaling=1`, `tcp_rmem`/`tcp_wmem` autotune.
- SACK (RFC 2018), DSACK for spurious retx.
- ECN (RFC 3168) — explicit congestion marks rather than drops. DCTCP/L4S build on it.
- TCP Fast Open (RFC 7413) — SYN carries data via cookie.
- BBRv2 adds loss + ECN response (fairness vs CUBIC); BBRv3 (draft-ietf-ccwg-bbr) further refined and standardizing.

**Grill-down.**
- L1: Nagle + delayed-ACK pathology — describe the 200 ms stall.
- L2: BDP math: 100 Gbps × 100 µs DC RTT = ? Tune `tcp_rmem` accordingly.
- L3: ECN vs PFC — both target loss avoidance; how do they layer? (ECN end-to-end; PFC per-hop. DCQCN combines both.)
- L4: TFO — what stops it from being universal? (Middlebox stripping; cookie management.)

**Resources.** Stevens Vol. 1; *TCP Tuning for HPC* (ESnet); BBR IETF drafts.

---

## 3.13 Network programming — sockets, epoll/kqueue, Node loop, Go netpoller, Tokio

**Concept.** Three I/O models: blocking, non-blocking + readiness (`epoll`, `kqueue`), completion-based (`io_uring`, IOCP). Frameworks:
- **Node.js:** libuv event loop, single JS thread, threadpool for FS/DNS.
- **Go netpoller:** integrates with goroutine scheduler; `netpoll_epoll` on Linux, parks goroutines on socket readiness.
- **Tokio (Rust):** work-stealing scheduler, `mio` wraps epoll/kqueue; emerging `tokio-uring`.

**Grill-down.**
- L1: Level- vs edge-triggered epoll. Pitfalls of edge-triggered.
- L2: Why does Node fall over on CPU-bound work despite great I/O? (Single JS thread.)
- L3: Goroutine M:N scheduling — what happens when 10k goroutines block on sockets? (Netpoller parks them; only runnable Gs occupy OS threads.)
- L4: io_uring vs epoll for a 1M-rps proxy — what changes in the syscall path?

**Cross-links.** OS (scheduling, context-switch cost), CN.

**Resources.** *The Linux Programming Interface* (Kerrisk); Go runtime source; tokio.rs docs.

---

## 3.14 Low-latency / HFT networking

**Concept.** Tick-to-trade under 10 µs end-to-end. Stack: colocation rack at the exchange; FPGA NIC or Solarflare/Mellanox in kernel-bypass mode; busy-poll thread pinned to isolated core with `isolcpus` + IRQ affinity; jitter-controlled BIOS (C-states off, P-states fixed). Wire format: native binary — NASDAQ **ITCH** for market data, **OUCH** for orders, CME **MDP**; **FIX** for institutional/cross-venue. Transport: **multicast** (PIM/IGMP) for market-data fan-out, TCP unicast for orders. Physical layer: microwave/laser links between Chicago (Aurora) and NJ (Carteret) — ~4 ms vs ~7 ms fiber.

**Industry application.** Industry write-ups put traditional kernel networking at 20–50 µs/transit and DPDK/RDMA at 1–5 µs. Cboe and Nasdaq ITCH-based feeds typically use UDP multicast with **MoldUDP64** framing (sequence numbers; gap-fill via TCP via a "GLIMPSE" snapshot service).

**Grill-down.**
- L1: Why multicast for market data? (1-to-many native, no per-subscriber serialization.)
- L2: MoldUDP64 gap recovery — how? (Sequence numbers; client requests retransmit via TCP rewinder service.)
- L3: NIC busy-poll vs interrupts — latency trade-off and CPU cost.
- L4: What is the slowest part of a kernel-bypass tick-to-trade pipeline? (Strategy logic + NIC TX queue; not the network.)
- L5: Microwave vs fiber Chicago↔NYC — why faster? (Straight line, lower refractive index in air vs glass.)

**Resources.** *Trading and Exchanges* (Harris); Nasdaq TotalView-ITCH spec; CME MDP docs; Databento microstructure guide.

---

## 3.15 Distributed-systems networking primitives

**Concept.**
- **Gossip:** SWIM (Cassandra, Consul), epidemic dissemination; O(log N) convergence.
- **Hinted handoff:** queue write for failed replica, deliver on recovery (Dynamo).
- **Read repair:** detect + heal divergence at read time.
- **Quorum:** R + W > N for consistency.

**Cross-links.** DDIA Ch. 5; Dynamo paper.

**Grill-down.**
- L1: Gossip — why does it scale better than master broadcast?
- L2: Read repair vs anti-entropy — when each?
- L3: Network partition + quorum — what consistency model can you still offer?

---

## 3.16 Consensus protocol networking — Raft, Paxos message patterns

**Concept.**
- **Raft:** RequestVote, AppendEntries (heartbeat + log replication). Leader → followers, fan-out. 1 RTT per write commit (majority ack).
- **Paxos (Multi-):** Prepare → Promise → Accept → Accepted. Higher message complexity; Multi-Paxos elects a stable leader, then resembles Raft.
- **EPaxos / Flexible Paxos:** trade quorum shapes for latency in geo-replicated settings.

**Industry application.** etcd, Consul, CockroachDB use Raft. Spanner uses Paxos. Anthropic/OpenAI cluster controllers usually layer on etcd.

**Grill-down.**
- L1: Raft commit latency in a 3-node cluster across regions of 80 ms RTT?
- L2: Why is leader election in Raft randomized timeout? (Avoid split votes.)
- L3: EPaxos no-leader — what does it buy and cost?

**Resources.** Diego Ongaro's Raft thesis; *In Search of an Understandable Consensus Algorithm* (USENIX ATC 2014); DDIA Ch. 9.

---

## 3.17 Data replication networking — WAL shipping, CDC, Kafka ISR

**Concept.**
- **WAL shipping (Postgres streaming repl):** primary streams WAL records over TCP; sync vs async.
- **CDC (Debezium, Maxwell):** parse WAL, publish to Kafka.
- **Kafka:** partition leaders, **ISR** (in-sync replicas) — a replica must catch up within `replica.lag.time.max.ms`. `acks=all` requires all ISR to ack. Replication uses a dedicated socket per broker pair; zero-copy `sendfile` from log to socket.

**Industry application.** Databricks/Snowflake use object-store-mediated replication (no peer-to-peer). Kafka at LinkedIn moves trillions of msgs/day; tuning `socket.send.buffer.bytes`, `replica.fetch.max.bytes` matters.

**Grill-down.**
- L1: Kafka `acks=1` vs `acks=all` — failure modes.
- L2: Min ISR — how does it interact with availability under network partition?
- L3: `sendfile` zero-copy — what kernel APIs avoid the userspace bounce? (`sendfile`, `splice`, `MSG_ZEROCOPY`.)

**Cross-links.** OS (zero-copy), DB (logical vs physical repl).

**Resources.** Kafka docs *Replication*; Postgres docs *Streaming Replication*.

---

## 3.18 Data-engineering: Snowflake & Databricks data plane

Memorize for a Databricks/Snowflake interview.

**Snowflake (Dageville et al., SIGMOD 2016).**
- Storage/compute separation: *"Snowflake separates storage and compute. The two aspects are handled by two loosely coupled, independently scalable services. Compute is provided through Snowflake's (proprietary) shared-nothing engine. Storage is provided through Amazon S3 … To reduce network traffic between compute nodes and storage nodes, each compute node caches some table data on local disk."*
- Wire protocol: *"S3 is a blob store with a relatively simple HTTP(S)-based PUT/GET/DELETE interface … S3 does, however, support GET requests for parts (ranges) of a file."* → range GETs let Snowflake fetch only needed columns from Parquet/PAX files.
- Caching: *"Each worker node maintains a cache of table data on local disk … the query optimizer assigns input file sets to worker nodes using consistent hashing over table file names … Whenever a worker process completes scanning its set of input files, it requests additional files from its peers, a technique we call file stealing."*
- PrivateLink (Snowflake docs): *"AWS PrivateLink is an AWS service for creating private VPC endpoints that allow direct, secure connectivity between your AWS VPCs and the Snowflake VPC without traversing the public internet."* Internal stages can also use S3 interface endpoints to keep load/unload off the public Internet.

**Databricks (Behm et al., *Photon*, SIGMOD 2022; Armbrust et al., *Delta Lake*, PVLDB 2020).**
- Decoupled storage: *"Databricks' platform decouples data storage from compute, allowing customers to choose their own low-cost storage provider (e.g., S3, ADLS, GCS) … Databricks accesses customer data using connectors between a compute service and the data lake. The data itself is stored in open file formats such as Apache Parquet."*
- Why CPU-bound now: *"low-level optimizations such as local NVMe SSD caching and auto-optimized shuffle have significantly reduced IO latency. … techniques such as data clustering, enabled by Delta Lake, allow queries to more aggressively skip unneeded data via file pruning."*
- Shuffle transport: *"For plans that end with a data exchange, Photon writes a shuffle file that conforms to Spark's shuffle protocol, and passes metadata about the shuffle file to Spark."* (Transport remains Spark's Netty block-transfer service; no publicly documented production RDMA shuffle.)
- PrivateLink (Databricks docs): two endpoints — a **workspace endpoint** "for REST API calls for both inbound and classic compute plane PrivateLink" and an **SCC relay endpoint** "specifically for secure cluster connectivity (SCC) between the compute plane and control plane."

**Grill-down.**
- L1: Why object storage as the durable layer? (Independent scaling, durability ~11 nines, cheap.)
- L2: A query reading 1 TB Parquet from S3 — what dominates latency? (First-byte latency, TLS, then bandwidth.)
- L3: Snowflake "file stealing" — what distributed problem does it solve? (Stragglers from skewed input partitioning.)
- L4: Spark shuffle on AWS — what is the network pattern? (All-to-all between executors; mappers write local disk + Netty block xfer on reduce fetch; "auto-optimized shuffle" coalesces small shuffle files.)
- L5: PrivateLink vs VPC peering — why does Snowflake mandate PrivateLink for enterprises? (Avoid CIDR overlap, unidirectional exposure of a SaaS endpoint, network-level isolation.)

**Resources.** Dageville et al. SIGMOD 2016; Behm et al. SIGMOD 2022 (*Photon*); Armbrust et al. PVLDB 2020 (*Delta Lake*); Snowflake docs `admin-security-privatelink`; Databricks docs `privatelink-concepts`.

---

## 3.19 ML inference networking — SSE, batching, KV cache, disaggregated prefill/decode

**Concept.**
- **Streaming responses:** all major LLM APIs use SSE over HTTP/1.1 or HTTP/2; tokens emitted as `event: token\ndata: {…}` lines.
- **Continuous batching (Orca, vLLM):** dynamic batch composition per step, not per request — improves throughput 5–20×.
- **Prefill vs decode:** prefill is compute-bound (one shot over the prompt); decode is memory-bandwidth-bound (autoregressive). Co-locating them on the same GPU causes decode latency variance.
- **Disaggregated prefill/decode (DistServe, Mooncake, llm-d, vLLM):** run on separate GPU pools; transfer KV cache over the network from prefill→decode. KV cache size = `2 × layers × kv_heads × head_dim × dtype` per token. For a Llama-3-70B-class model (80 layers, 8 KV heads, head_dim 128, FP16) the math is `80 × 8 × 128 × 2 × 2 = 327,680` bytes/token, i.e. ≈ 320 KB/token; a 4k-token prompt → ~1.3 GB transfer.
- **Transport:** NIXL (NVIDIA Inference Xfer Library), UCX over NVLink/RDMA/TCP; vLLM router uses ZeroMQ. With NVLink/RDMA, KV transfer completes in low-ms; with TCP, hundreds of ms.

**Industry application.** Per BentoML LLM Inference Handbook: *"Several open-source frameworks and projects are actively exploring PD disaggregation, including SGLang, vLLM, Dynamo, and llm-d."* Per llm-d 0.5 release notes: hierarchical KV offloading + UCCL networking + cache-aware LoRA routing. Per JarvisLabs: Meta's internal disaggregated implementation uses custom connectors for KV cache transfer.

**Grill-down (the Anthropic/OpenAI zone).**
- L1: Why SSE for LLM streaming and not WebSocket?
- L2: Continuous batching — what is the networking implication? (Heterogeneous request lengths in one batch → return one stream at variable rate; need careful HTTP/2 flow control.)
- L3: KV cache transfer math for Llama-3-70B with TP=8, 4k-prompt → how many GB and what is the wall time at 200 Gb/s RDMA vs 25 Gb/s TCP?
- L4: When does PD disaggregation *hurt*? Per BentoML: *"For shorter prompts or when the decode engine has a high prefix cache hit, running prefill locally on the decode worker is often faster."* Also when KV transport is TCP and the transfer dominates the TTFT budget.
- L5: Multi-tenant serving — how do you isolate noisy neighbors at the network layer? (Per-tenant QoS via tc/eBPF, separate inference pools, token-bucket per API key.)
- L6 (deep): KV cache *paging* (PagedAttention/vLLM) — what does it imply for KV transfer? (Non-contiguous physical layout → scatter-gather DMA; NIXL handles this.)

**Cross-links.** ML infra, OS (HugePages for KV), CN ↔ GPU (GPUDirect RDMA writes directly into HBM).

**Resources.** vLLM `disagg_prefill` docs; *DistServe* (OSDI 2024); *Mooncake* (Moonshot AI); llm-d.ai docs; Perplexity Research blog "Disaggregated Prefill and Decode."

---

# How different companies grill differently

| Company | What they emphasize | Signature questions |
|---|---|---|
| **Anthropic / OpenAI / DeepMind** | Training fabric + inference serving. RDMA, NCCL internals, collective topology, disaggregated PD, KV transfer, multi-tenant isolation. | "Wire a 1024-GPU H100 cluster for a 70B-param model. Where does each parallelism axis live?" / "How does NCCL pick ring vs tree?" / "Math out KV cache transfer for Llama-70B." |
| **Google** | Datacenter fabric, BGP, Jupiter/Aquila, Maglev, anycast, BBR. Long history of first-principles transport questions. | "Design a global LB that survives a region loss in 30 s." / "Why did Jupiter move off Clos?" / "BBR vs CUBIC trade-off." |
| **Meta** | RoCE for AI, scale ops, BGP at the edge, video streaming. SIGCOMM 2024 RoCE paper is fair game. | "Why does ECMP fail for AI training and what did you do?" / "Design Live's origin shielding." |
| **Netflix** | CDN architecture (Open Connect), adaptive bitrate, regional caching, microservice resilience. | "How does an ISP-embedded OCA decide what to cache?" / "Trace a 4K HDR play from click to glass." |
| **Amazon (AWS)** | VPC, PrivateLink, NLB/ALB, Global Accelerator, cross-region. Detail-orientation. | "Two VPCs overlap CIDRs — connect them." / "Difference between NLB and ALB at L4?" |
| **Apple** | Privacy-first networking, iCloud Private Relay (built on QUIC), TLS, MASQUE. | "Design a 2-hop proxy that hides client IP from origin." |
| **Cloudflare** | Anycast, eBPF/XDP, QUIC/HTTP3, DDoS at scale, RPKI. | "How does anycast TCP not break?" / "Walk me through XDP DDoS mitigation." |
| **Databricks / Snowflake** | Compute/storage separation, S3 semantics, shuffle, PrivateLink, caching tiers. | "S3 range GETs and Parquet column pruning — explain the network savings." / "Spark shuffle network pattern." |
| **Jane Street / Citadel / Two Sigma / Hudson River** | Tick-to-trade, kernel bypass, multicast, jitter, NUMA, lock-free. OCaml/C++ idioms. | "Where is your tick-to-trade slowest and how would you measure it?" / "Multicast gap recovery." |
| **Goldman / JPM / McKinsey Digital** | Enterprise patterns: VPN, ExpressRoute/Direct Connect, compliance, zero-trust. More architectural than low-level. | "Design a cross-region trading system with FINRA controls." / "VPN vs SD-WAN trade-offs." |

---

# 12-Week Study Plan

| Week | Focus | Goal artifact |
|---|---|---|
| 1 | OSI/TCP-IP, L1/L2, IP/CIDR/NAT. KR Ch. 1–4 fast. | Subnet math drills; build a CIDR calculator. |
| 2 | Routing + BGP fundamentals; RPKI. | FRR/BIRD lab; announce routes between two AS in containerlab. |
| 3 | **TCP deep dive + TIME_WAIT + congestion control.** Read BBR (Cardwell). | CS144 labs 1–3 (build TCP receiver/sender). |
| 4 | DNS + HTTP/1.1 vs 2 vs 3 + TLS 1.3. HPBN Ch. 4, 11–12. | Run `dig +trace`; profile a TLS handshake; enable H3 on local nginx-quic. |
| 5 | Load balancing + Maglev paper + Envoy basics. | Build 2-tier LB on Envoy with consistent hashing; benchmark vs RR. |
| 6 | CDN + Anycast + Cloudflare blog dive. | Trace Netflix manifest fetch; diagram cache hierarchy. |
| 7 | Service mesh, observability, OpenTelemetry. *Istio in Action* skim. | Spin up Cilium service mesh on kind/minikube; capture mTLS handshake. |
| 8 | **Kernel bypass: DPDK + XDP + io_uring + Cilium.** | Run XDP DDoS-drop sample on a VM; bench DPDK l2fwd. |
| 9 | **RDMA + RoCE + NCCL.** Read Meta SIGCOMM 2024 + *Demystifying NCCL*. | Two-GPU `nccl-tests` AllReduce; explain the topology log. |
| 10 | GPU topology + AI fabric + Jupiter. Read SIGCOMM 2022. | Diagram an H100 SuperPOD + map TP/PP/DP onto the network. |
| 11 | Tail latency, TCP optimization, low-latency / HFT, multicast. | Tune `tcp_*` sysctls; build a UDP multicast pub/sub. |
| 12 | DistSys + DB + ML-infra networking (Snowflake, Databricks, disaggregated PD). Read Photon, Snowflake, *Tail at Scale*. Mock interviews. | Whiteboard 5 system designs end-to-end. |

**Daily routine.** 90 min reading/coding + 30 min one grill-down topic out loud. Track wins in spaced-repetition (Anki) for: RFC numbers, port numbers, sysctl names, BDP math, NCCL primitives.

---

# Master Resources

**Books.** KR, Stevens Vol. 1, HPBN (free), DDIA, *SRE Book* + *SRE Workbook* (free), *Computer Systems: A Programmer's Perspective* for OS↔CN, *Cloud Native Patterns* (Davis).
**Papers.** BBR (Cardwell, ACM Queue 2016); QUIC (Langley et al., SIGCOMM 2017); *Jupiter Rising* (SIGCOMM 2015) + *Jupiter Evolving* (SIGCOMM 2022); *Maglev* (NSDI 2016); Meta RoCE (SIGCOMM 2024); *Demystifying NCCL* (arXiv 2507.04786); Snowflake (SIGMOD 2016); *Photon* (SIGMOD 2022); *Delta Lake* (PVLDB 2020); *The Tail at Scale* (CACM 2013); Dapper; *The Datacenter as a Computer* (Barroso & Hölzle).
**Courses.** Stanford CS144 (do the labs); MIT 6.829; CMU 15-441; Brendan Gregg's eBPF talks.
**Engineering blogs.** Cloudflare, Netflix Tech, Meta Engineering, Google Cloud, AWS Architecture, High Scalability, Databricks Engineering, Snowflake Engineering.
**Docs.** Linux `Documentation/networking/*`, key RFCs (793/9293 TCP, 9000 QUIC, 9114 HTTP/3, 8446 TLS 1.3, 4271 BGP-4, 9113 HTTP/2, 5681 Reno, 9438 CUBIC), Envoy/Istio/Cilium/NCCL docs.

---

# Caveats and Honest Notes

- **HTTP/3 share is not "a third."** Per Cloudflare *Radar 2025 Year in Review*, HTTP/3 is 21% of Cloudflare requests, HTTP/2 is 50%, HTTP/1.x is 29% — shares largely unchanged from 2024. Earlier reports peaked closer to 28% in May 2023.
- **BBRv3** is still in IETF process (`draft-ietf-ccwg-bbr-05` in 2026). Production Linux typically ships BBRv1; BBRv2 was internal-only at Google for years; BBRv3 fairness work is ongoing.
- **No publicly documented RDMA shuffle at Databricks.** The Photon paper says shuffle conforms to Spark's protocol (Netty block transfer). If asked, say so explicitly rather than speculating.
- **NCCL algorithms are version-dependent.** NVLS/CollNet support depends on hardware (NVSwitch 3+) and NCCL version (≥ 2.19 for full set).
- **0-RTT** is replay-vulnerable; never assume it is enabled for non-idempotent ops without explicit reasoning. Cloudflare ships it via `ssl_early_data`; sensitive endpoints should return HTTP 425.
- Some sub-microsecond HFT latency numbers in vendor blogs are aspirational; real tick-to-trade is dominated by strategy logic, not network, after the first round of optimization.
- BBR's headline numbers come from the original paper: per Cardwell et al., *"BBR reduces median RTT by 53 percent on average globally, and by more than 80 percent in the developing world."* These were YouTube measurements circa 2016; current numbers will differ.
- Netflix Open Connect's "~95% via direct ISP connections" figure is from APNIC (June 2018) quoting Netflix Open Connect; precise current share is not published every year, but order of magnitude is stable.