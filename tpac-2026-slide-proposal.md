# TPAC 2026 slide proposal: Using QUIC to traverse NATs

Working proposal for a ten-minute talk. This document plans the slides and their intent; it is not a speaker script. The current iteration has twelve visible slides and one hidden backup in `index.html`, with editable diagrams and incremental reveals.

**Proposed scope:** show the setup responsibilities of QUIC over ICE, then ask “What if QUIC could do it all?” Recap RFC 9000 connection migration and preferred address. Introduce two ingredients: address advertisement, which also serves independent public-server use cases, and server-side probing coordinated over an assumed relayed connection. Bring them together as hole punching inside QUIC, then compare the startup sequence and client-driven policy with full ICE. Return to the initially skipped setup question near the end: should there be a common relay setup mechanism, using rendezvous or CONNECT-UDP Listen (now Bind)? Close with the reference recap.

**Timing:** twelve visible slides, including the comparison table and closing references: 7:40 of planned delivery and 2:20 of buffer. “What if both peers could open a path?” remains a hidden backup. Slides 2 and 3 are background that can be covered in about 15 seconds combined if Jan-Ivar has already established it, saving 1:00. Allow 0:45 for walking through the comparison table.

## Drafts behind the concept

Introduce two distinct ingredients with a one-way dependency: server-side probing uses advertised addresses, while address advertisement has independent uses. Each ingredient slide links its draft with a compact title and author names at the bottom. The closing recap also credits the authors. The relay setup slide links both options. The title slide names only the presenter.

| Draft at the IETF | Authors | Contribution to the talk |
| --- | --- | --- |
| [UDP Rendezvous over HTTP][rendezvous] | Marten Seemann | One possible standardized starting point, introduced as an open question on slide 11. |
| [QUIC Alternative Server Address Frames][alternative-address] | Marco Munizaga, Marten Seemann | Ingredient 1: advertise addresses and preferences, illustrated by a public server changing its preferred address on slide 7. |
| [Using QUIC to traverse NATs][nat-traversal] | Marten Seemann, Eric Kinnear | Ingredient 2: enable server-side probing of new candidate paths, coordinate attempts, then migrate to a suitable direct path, on slides 8–9. |

Also include [QUIC Address Discovery][address-discovery], by Marten Seemann and Christian Huitema, on the closing slide. It lets an endpoint learn the address observed by its QUIC peer on a path and can contribute to candidate gathering. Link to the working-group draft, `draft-ietf-quic-address-discovery`.

The relay setup discussion also includes [CONNECT-UDP Listen][connect-udp-listen], by David Schinazi and Abhijit Singh. The draft now calls the mechanism CONNECT-UDP Bind and is titled *Proxying Bound UDP in HTTP*.

Keep one connected story. The core traversal sequence assumes an initial relayed connection without prescribing its setup. Rendezvous is introduced later as a candidate, not as a dependency of the traversal mechanism.

## Agreed direction

- **Build from familiar QUIC:** recap a client moving from Wi-Fi to cellular, then the server's preferred address in the handshake, before introducing the relayed peer connection. Keep the separate “What if both peers could open a path?” slide as a hidden backup. Distinguish client-initiated migration from path validation, which both endpoints already perform.
- **Lead message:** one connected story: use an established relayed QUIC connection to coordinate NAT traversal, then use QUIC's path validation and migration mechanisms to move that same connection to a direct path. Reusing QUIC's mechanisms and maintaining the connection throughout are parts of the same explanation.
- **Defer initial setup:** start the core explanation with a working relayed QUIC connection. Do not explain authentication, peer discovery, or relay setup before probing and migration.
- **Two ingredients:** slide 7 shows address advertisement working independently with a public server. Slide 8 introduces the relayed peer topology and adds server-side probing of new candidate paths, with coordinated timing, using those addresses. Address advertisement does not require the hole-punching mechanism.
- **Close the loop:** return to the assumed initial connection on slide 11 and ask what, if anything, should be standardized. Present UDP Rendezvous over HTTP and CONNECT-UDP Listen (now Bind) as options.
- **Service layout:** any application authentication/signaling service and the packet relay remain separate. The core diagrams show laptop peers and a public relay server with a server-rack icon.
- **Comparison:** use three short rows for application data, pairing and timing, and server responsibilities. Keep handshake details and qualifications in this planning document. Compare with full ICE, without claiming a measured performance advantage.
- **Implementation and status:** keep the presentation focused on the mechanism. There is no implementation experience or IETF status update to add to this proposal.

## Visual direction

Use [Tailscale’s 2020 NAT traversal article](https://tailscale.com/blog/how-nat-traversal-works), its [later NAT traversal illustrations](https://tailscale.com/blog/nat-traversal-improvements-pt-1), and [Cloudflare’s network diagrams](https://developers.cloudflare.com/reference-architecture/diagrams/network/protect-public-networks-with-cloudflare/) as references for clarity: generous whitespace, simple network boundaries, restrained colors, short labels placed beside the relevant path, and successive views that change only the important state. Create original diagrams with our own geometry, icons, palette, and wording.

On diagram slides, the main diagram should occupy roughly three quarters of the slide. Use laptop icons for peers and server-rack icons for public servers. Keep closed, lightly shaded network regions with darker firewall bands and lighter openings that represent outbound state. Omit repeated NAT/firewall labels. The migration recap retains Wi-Fi and cellular symbols, inspired by [MPTCP’s compact path labels](https://www.mptcp.dev/) and [phone status icons](https://support.apple.com/guide/iphone/learn-the-meaning-of-iphone-status-icons-iphef7bb57dc/ios). On slides 8–9, use three plain interface points per peer, inside the network box between the laptop and firewall. The top point connects to the relay; the lower two represent candidate paths. Keep peer A on the left and peer B on the right. Introduce the relay by name on slide 8 and keep it above the peers. Align the endpoint geometry across the coordination and migration states. Introduce specific relay setup options only on slide 11.

On slide 8, show the frame exchange as two parallel arrow routes through the relay: green PUNCH_REQUEST from client to server, and purple ALTERNATIVE_ADDRESS from server to client. Put an arrowhead at each receiving peer and label the routes with frame names in matching colors. Both routes represent messages on the same relayed QUIC connection. Slide 7 uses a client and public server to demonstrate address advertisement independently of hole punching, with ALTERNATIVE_ADDRESS on the working connection and a preferred alternative address. Use orange dashed probes in both directions on each lower pair, emphasizing the server’s probes. Leave gaps beside the interface points and devices. On the next slide, mute unused paths when application data moves to the bottom path. Keep labels short, avoid packet layouts and address tables, and omit “Public Internet” labels.

Where a slide has successive states, use one or two reveals and keep the established context visible. Each final state must also make sense in an exported PDF. These are changes within a slide, not additional slides in the timing budget.

**Draft references:** each ingredient slide includes a linked draft title and its authors in the footer. Keep author names on the closing slide as well. The relay setup slide has two linked options with author names beneath. No draft-header screenshots are needed.

## Slide 1 — Using QUIC to traverse NATs

**Time:** 0:30 · cumulative 0:30

**Show:** the official TPAC cover artwork with the talk title and Marten's name. Keep the diagrams on the content slides that follow.

**On-slide words:** “Using QUIC to traverse NATs” and “Marten Seemann.”

**Intent:** introduce the broader peer-to-peer QUIC concept, drawing on several complementary drafts. QUIC can carry data through a relay while the peers use its path mechanisms to establish a direct route.

## Slide 2 — How hole punching works

**Time:** 0:45 · cumulative 1:15

**Show:** two peers behind separate NAT/firewall boundaries, with three successive states of a simple successful case. First, A sends toward B and the packet is blocked at B's boundary. A lighter opening is already visible at A. Next, B also sends outbound toward A and a lighter opening appears at B. Finally, packets can pass in both directions through the state created by outbound traffic.

**On-slide words:** “Peer A,” “Peer B,” and “Blocked” in the initial state. Let the arrows and lighter firewall openings show the changes; explain who sends and when traffic can pass orally.

**Intent:** outbound packets from both sides can create the NAT and filtering state needed for a direct path. This is a connectivity attempt whose success depends on the network, rather than a guarantee that every pair of peers can connect directly.

**Diagram note:** assume the peers have candidate addresses to try; later slides explain coordination. Illustrate a successful mapping/filtering case without suggesting all NATs behave identically or that packets require precisely synchronized transmission.

## Slide 3 — Run ICE, then QUIC

**Time:** 0:30 · cumulative 1:45

**Show:** a small “Join a call” box, a much larger ICE box, and a small QUIC handshake box. Inside ICE, stack four points vertically: address discovery, candidate exchange, connectivity checks, and path selection. Put a short explanation beneath each point. Include local addresses, STUN/TURN, application signaling, and the choice of a direct or TURN path. Do not connect the internal points with sequential arrows.

**On-slide words:** “Join a call,” “ICE,” “Address discovery,” “Candidate exchange,” “Connectivity checks,” “Path selection,” “Direct path or TURN relay,” and “QUIC handshake.” Omit the application-intent subtitle and the caption below the diagram.

**Intent:** there are several setup responsibilities before QUIC can use the selected ICE path. Transition to the QUIC mechanisms we can reuse, starting with a familiar connection-migration example.

**Delivery note:** joining a call is an example of application intent. This is the QUIC-over-ICE baseline, not the full WebRTC procedure or a claim about Jan-Ivar's proposal. If he has covered this background, move quickly to the next slide.

**Accuracy note:** candidate exchange uses application signaling, whose protocol ICE leaves open. TURN candidates may be gathered and checked alongside direct candidates. Gathering, exchange, and checks can overlap with [Trickle ICE](https://www.rfc-editor.org/rfc/rfc8838.html#section-1); the points are not a literal serial timeline or evidence of worse performance. [Draft background][draft], [ICE overview][ice].

## Slide 4 — What if QUIC could do it all?

**Time:** 0:05 · cumulative 1:50

**Show:** only the question, centered in large type on the TPAC background.

**On-slide words:** “What if QUIC could do it all?”

**Intent:** pivot from the ICE setup machinery to QUIC's existing path mechanisms and the two proposed ingredients. This is a motivating question, not a claim that QUIC replaces application discovery or authentication.

## Slide 5 — RFC 9000: QUIC connection migration

**Time:** 1:00 · cumulative 2:50

**Show:** a laptop client inside a closed NAT/firewall region on the left, and a public QUIC server on the right. Place both interface icons inside the client’s network box, with Wi-Fi arcs beside the upper path and cellular signal bars beside the lower path. Include the text labels “Wi-Fi” and “Cellular” here to introduce both symbols. Use three states: the established Wi-Fi path; both endpoints validating the cellular path together; and application data moving to cellular on the same connection. Show validation with two straight, parallel, bidirectional lanes, one for PATH_CHALLENGE and one for PATH_RESPONSE. Keep the original path greyed out in the final state, without its label. The public server has no NAT or firewall boundary.

**On-slide words:** “QUIC client,” “QUIC server,” “Wi-Fi,” “Cellular,” “Established path,” “PATH_CHALLENGE,” “PATH_RESPONSE,” and “Application data.”

**Intent:** give participants a concrete migration picture before using that mechanism for NAT traversal. Show the probes in both directions, with the server sending its own challenge as well as responding to the client.

**Accuracy note:** the bidirectional arrows summarize both exchanges together without prescribing timing or packet order. Challenge and response frames can share a packet, and probing before moving application data is optional. The server keeps its address in this example. Do not describe RFC 9000 as forbidding server path validation. [Path validation][validation], [connection migration][migration].

## Slide 6 — RFC 9000: preferred address

**Time:** 0:35 · cumulative 3:25

**Show:** the same client and public server, with an initial anycast address and a preferred unicast address at the server. The client first reaches the server through the shared anycast address. The handshake announces that server's unicast address. Show validation with the same two parallel, bidirectional arrows as the recap: orange challenges and green responses. Use one static diagram.

**On-slide words:** endpoint labels, “Preferred address in handshake,” “Anycast address,” “Unicast address,” “Validate, then migrate,” and “No address updates after the handshake.”

**Intent:** illustrate a key motivation for preferred address: move from a shared anycast address to the serving endpoint's unicast address for connection stability. Establish this RFC 9000 baseline before ingredient 1 extends it to later address updates. [RFC 9000 section 9.6][preferred-address].

## Slide 7 — Ingredient 1: address advertisement

**Time:** 0:45 · cumulative 4:10

**Show:** a client behind a NAT and a public server without a NAT boundary, using the positions from the migration recap. Keep the client's Wi-Fi interface fixed. A solid green connection reaches address A at the server. The server advertises B as preferred over that connection; an orange dashed path leads from the client to address B on the same server, showing validation before migration. This is one static diagram without a relay or coordinated hole-punching attempts.

**On-slide words:** “QUIC client,” “QUIC server,” “Address A,” “B (preferred),” “ALTERNATIVE_ADDRESS,” “Probe, then migrate,” and “Independent use: server address changes.” Link [QUIC Alternative Server Address Frames][alternative-address] and name Marco Munizaga and Marten Seemann beneath it.

**Intent:** introduce Marco's draft as the first ingredient, useful on its own. It extends the handshake-only preferred address from slide 6: a public server can update its addresses and preferences during a connection, allowing the client to validate another address and migrate. The example illustrates the draft's multihomed-server use case; coordinated hole punching is not a dependency.

**Diagram note:** A and B are addresses of the same QUIC endpoint. The client initiates validation and migration; preferences are advisory, and learning an address does not establish reachability. The orange arrow summarizes validation rather than showing every packet. Address discovery and NAT traversal are outside this extension's scope. The next slide reuses address advertisement as an input to server-side probing. [Alternative addresses, introduction and sections 4–5](https://datatracker.ietf.org/doc/html/draft-munizaga-quic-alternative-server-address-02#section-1).

## Slide 8 — Ingredient 2: server-side probing

**Time:** 1:00 · cumulative 5:10

**Show:** two peers behind NATs, with three plain interface points spaced vertically inside taller peer boxes and a relay server above. Two parallel arrow routes share the top relayed connection: green PUNCH_REQUEST arrows go from client through relay to server, and purple ALTERNATIVE_ADDRESS arrows go from server through relay to client. Label each route with its frame name in the matching color, without sender/recipient prose. Each lower pair has orange probes in both directions, with the server's probes emphasized. Omit the bottom narration caption and use that space to separate the interfaces. Keep one static diagram and a linked draft title with author names below it.

**On-slide words:** peer and QUIC role labels, “Relay server,” “Relayed QUIC connection,” “ALTERNATIVE_ADDRESS,” “PUNCH_REQUEST,” and “QUIC path probes.” Link [Using QUIC to traverse NATs][nat-traversal] and name Marten Seemann and Eric Kinnear in the footer.

**Intent:** introduce Eric's draft as the second ingredient: the server can probe a new candidate path toward the client before receiving a direct-path packet. The client repeatedly selects pairs and requests attempts over the existing connection. Timing coordination makes the probing periods overlap, without synchronized clocks or exact simultaneous transmission. This ingredient depends on the advertised addresses from the first, which also has uses of its own.

**Accuracy note:** RFC 9000 already allows server path validation. The added capability is arranging server probes toward selected candidate addresses when both sides may need to send before either is reachable. Timing coordination supports that capability.

**Delivery note:** assume the relayed connection exists without prescribing its setup. It carries both coordination and application data. The client chooses which pairs to attempt; the two shown are examples, not the full set of possible pairings or a required schedule. Returning responses validate a suitable path. The next slide keeps the same interfaces and shows migration to one lower path. Candidate gathering remains separate: QUIC Address Discovery can supply an observed address, without guaranteeing reachability from another peer. Apart from ALTERNATIVE_ADDRESS and PUNCH_REQUEST, omit frame names, punching rounds, grants, and connection IDs. [Coordinated probing in the local draft][draft], [QUIC path validation][validation], [address discovery][address-discovery].

## Slide 9 — Hole punching inside QUIC

**Time:** 0:30 · cumulative 5:40

**Show:** keep the three plain interfaces per peer from the preceding diagram. The first state shows data on the top relayed path and QUIC path validation on both lower candidate paths. One reveal shows migration using the bottom path, with the relay and the other candidate path muted. Keep the endpoints fixed throughout.

**On-slide words:** “QUIC path validation” before the reveal and “QUIC connection migration” afterward, alongside peer and QUIC role labels.

**Intent:** hole punching uses QUIC's own path mechanisms inside an existing connection. Coordination and application data can flow over the relay while paths are tested; successful validation enables migration without another handshake.

**Diagram note:** this describes the outcome of the initial traversal attempt. Do not promise automatic recovery through the relay after every later direct-path failure, or imply that simultaneous use of both paths is required. [Draft introduction][draft], [QUIC migration][migration].

## Slide 10 — ICE and QUIC hole punching

**Time:** 0:45 · cumulative 6:25

**Show:** a static three-column table with three rows, large type, and generous padding. Keep every row visible. Use short cells and horizontal rules, with no subtitle, qualification line, or additional diagrams.

| | ICE | QUIC hole punching |
| --- | --- | --- |
| Application data | After connectivity checks | During hole punching |
| Pairing & timing | Both endpoints run checklists | Client decides |
| Server | Pair candidates; schedule and run checks | Advertise addresses; probe on request |

**Intent:** distinguish startup dependencies and ownership of the pairing algorithm. The client receives server candidates, pairs them with its own, and selects which pairs to attempt and when. The server advertises candidates and carries out authorized requests, allowing changes to pairing policy without a matching algorithm on the server.

**Accuracy notes:** the latency row compares order of work, not measured performance. Initial relay setup remains outside this sequence. ICE needs a usable path, not completion of every check; Trickle ICE overlaps gathering and checking. Compare full agents, not ICE-lite. ICE's controlling role governs nomination, while both full agents run checklists. ICE also permits local preferences and nomination criteria. The QUIC server retains authorization, limits, state, and validation work; the client still distinguishes candidate bases. [Local draft, pairing and coordinated probing][draft], [RFC 8445 sections 6.1 and 8.1.1](https://www.rfc-editor.org/rfc/rfc8445.html#section-6.1), [Trickle ICE](https://www.rfc-editor.org/rfc/rfc8838.html#section-1).

## Slide 11 — Standardize the initial relayed path?

**Time:** 1:00 · cumulative 7:25

**Show:** return to the initially skipped connection setup. Reuse the peer/relay geometry and solid green relayed QUIC path. Label the public node “Relay server.” Show the common topology without Join/Listen arrows or transport labels on the individual legs. Beneath the diagram, place links to [UDP Rendezvous over HTTP][rendezvous] and [CONNECT-UDP Listen (now Bind)][connect-udp-listen], with their author names.

**On-slide words:** “Relay server,” “Relayed QUIC connection,” “UDP Rendezvous over HTTP,” and “CONNECT-UDP Listen (now Bind),” plus peer labels and author credits. The title poses the standardization question; explain the setup differences orally.

**Intent:** close the loop: should there be a shared way to obtain the initial relayed connection, and which relay mechanism suits the browser use case? Neither option is required by the preceding traversal sequence.

**Discussion prompts:** which parts of the initial setup should be common across applications? Should it use rendezvous or CONNECT-UDP Listen/Bind? What should remain the application's responsibility?

**Scope note:** the common diagram does not imply identical setup or encapsulation. Rendezvous associates two HTTP clients at one proxy using an ID. Bind allocates a public UDP address and port at the proxy; another peer can send UDP there, while the requesting peer exchanges payloads with the proxy over HTTP. Peer discovery, signaling, and end-to-end authentication still need an application design. Keep the application authentication/signaling service separate from the packet relay. [Rendezvous draft][rendezvous], [CONNECT-UDP Listen/Bind sections 2, 7, and 8][connect-udp-listen], [ICE overview][ice].

## Slide 12 — Thank you / draft references

**Time:** 0:15 · cumulative 7:40

**Show:** four spacious references with linked titles and author names: [UDP Rendezvous over HTTP][rendezvous] (Marten Seemann), [QUIC Alternative Server Address Frames][alternative-address] (Marco Munizaga and Marten Seemann), [Using QUIC to traverse NATs][nat-traversal] (Marten Seemann and Eric Kinnear), and [QUIC Address Discovery][address-discovery] (Marten Seemann and Christian Huitema). Each link opens its IETF Datatracker page.

**On-slide words:** “Thank you,” the four titles and author lists, and short labels for their roles: one possible starting point, ingredient 1 (advertise addresses), ingredient 2 (server-side probing), and observed-address discovery.

**Intent:** give participants the source documents and acknowledge the collaborators behind the concept, including Christian's contribution to address discovery. Leave this slide up for discussion, returning to the relay setup question if useful.

**Visual note:** use editable title/author text in the current version. Full abstracts would make this four-reference slide too dense; reserve any header-and-abstract screenshot treatment for individual draft introductions.

## Hidden backup — What if both peers could open a path?

**If restored:** allow 0:45. Hidden from navigation, overview, and print output.

**Show:** replace the public server with a second laptop behind NATs/firewalls. Keep a solid green original path between the laptops, with Wi-Fi icons inside the two peer boxes near each end. Draw outbound QUIC probe arrows below it, with cellular signal bars inside the two peer boxes near each end, toward the selected remote addresses. Explain the need for candidate addresses and coordination orally.

**On-slide words:** peer and QUIC role labels, “Original path,” and “QUIC path probes.” Omit narration captions above and below the diagram.

**Intent:** extend the recap to a setting where neither peer can necessarily receive before sending. This resembles ICE's candidate-pair checking and raises the question of where to carry the coordination.

**Accuracy note:** the addition is coordinated probing of new candidate paths for NAT traversal. Both endpoints already perform path validation in RFC 9000. The shaded areas group private networks; the interfaces need not share one NAT. The orange arrows depict attempts, not guaranteed delivery; the green path is already working. [RFC 9000 section 8.2][validation], [draft source][draft].

## Delivery and scope notes

- **If Jan-Ivar covers the background:** show slides 2 and 3 briefly, acknowledge the shared mechanics, and spend the saved time explaining slides 5–9 or leave more room for discussion. No alternate deck is needed.
- **If time runs short:** shorten the background on slides 2 and 3 and keep the relay setup discussion on slide 11. The migration recap on slide 5 establishes the picture used throughout slides 7–9.
- **No dedicated MoQ slide in this proposal:** the traversal mechanism does not depend on a media application. MoQ can be mentioned as a motivating application if Marten wants it, without adding a separate workstream/status survey.
- **Keep protocol details for discussion:** punching rounds/grants, connection-ID lifetimes, frame layouts, candidate-pair algorithms, and multipath behavior are outside these ten minutes.
- **Keep the starting point open:** neither rendezvous nor CONNECT-UDP Listen/Bind is a dependency of the traversal sequence. They are options raised in the closing discussion.
- **Treat maturity precisely:** this concept combines work at different stages. The NAT traversal explanation follows the current local revision, which still has unfinished amplification-mitigation text; do not present browser security or abuse prevention as settled.

## Sources for preparing the slides

The four drafts behind the traversal concept also appear on the closing slide. Draft titles, authors, and public versions were checked on 8 October 2026; the alternative-address draft's independent use case was checked against revision -02 on 23 October 2026. The additional CONNECT-UDP Listen/Bind option was checked against revision -16 on 27 October 2026. The detailed NAT traversal sequence follows the current local draft, including its unsubmitted edits.

- [Draft source repository][draft]: abstract, introduction, ICE background, candidate exchange, coordinated probing, and security considerations.
- [UDP Rendezvous over HTTP][rendezvous]: Marten Seemann, published revision -00, 16 August 2026. Relayed UDP between authenticated proxy clients; no public UDP address allocation. Authentication methods and rendezvous-ID distribution remain out of scope.
- [Proxying Bound UDP in HTTP][connect-udp-listen]: David Schinazi and Abhijit Singh, revision -16, 24 August 2026. CONNECT-UDP Listen is now named Bind. The relay setup slide links this option alongside rendezvous.
- [QUIC Alternative Server Address Frames][alternative-address]: Marco Munizaga and Marten Seemann, published revision -02, 25 September 2026. Server address advertisement; address discovery and hole punching are separate mechanisms.
- [Using QUIC to traverse NATs][nat-traversal]: Marten Seemann and Eric Kinnear. The public page currently points to expired revision -02 from 3 March 2024, so it does not yet reflect the local rewrite. Keep the stable IETF link, and check the published revision before capturing a header screenshot for the final deck.
- [QUIC Address Discovery][address-discovery]: Marten Seemann and Christian Huitema, working-group revision -01, 15 August 2026. Use this successor to `draft-seemann-quic-address-discovery` for the closing reference.
- [RFC 8445, overview of ICE][ice]: candidate gathering, connectivity checks, and nomination.
- [RFC 9000, path validation][validation] and [connection migration][migration]: the existing QUIC mechanisms being reused.
- [Tailscale visual reference](https://tailscale.com/blog/nat-traversal-improvements-pt-1): visual inspiration only; no Tailscale deployment statistics or product architecture are imported into this proposal.

[draft]: https://github.com/marten-seemann/draft-seemann-quic-nat-traversal
[rendezvous]: https://datatracker.ietf.org/doc/draft-seemann-masque-connect-udp-rendezvous/
[connect-udp-listen]: https://datatracker.ietf.org/doc/draft-ietf-masque-connect-udp-listen/
[alternative-address]: https://datatracker.ietf.org/doc/draft-munizaga-quic-alternative-server-address/
[nat-traversal]: https://datatracker.ietf.org/doc/draft-seemann-quic-nat-traversal/
[address-discovery]: https://datatracker.ietf.org/doc/draft-ietf-quic-address-discovery/
[ice]: https://www.rfc-editor.org/rfc/rfc8445.html#section-2
[validation]: https://www.rfc-editor.org/rfc/rfc9000.html#section-8.2
[migration]: https://www.rfc-editor.org/rfc/rfc9000.html#section-9
[preferred-address]: https://www.rfc-editor.org/rfc/rfc9000.html#section-9.6
