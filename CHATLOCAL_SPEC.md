# ChatLocal product and engineering specification

Status: proposed implementation baseline; not a release.

## Decisions
- macOS first, retain Node built-ins and existing MCP tools until measured evidence warrants a rewrite. No mandatory Docker, Electron or VM.
- Native menu-bar companion (Swift/AppKit), local daemon and separately bound control/remote listeners. Start/stop/pause from menu bar, no terminal required after installation. No launch at login without opt-in.
- Tailscale Funnel as first stable endpoint candidate; named Cloudflare Tunnel as alternative. Quick tunnels only for temporary testing. Funnel is public, not a private access control. Never automatically switch endpoint or provider without telling the operator.
- Explicit local approval for each new OAuth client, S256 PKCE, exact redirects, revocable credentials; no unauthenticated authorization or legacy token bypass. Separate local admin secret and listener. Do not expose dashboard via tunnel.
- Preserve native host shell as optional elevated mode. File-tool jail does not sandbox shell. No claims of OS isolation. Default shell off, file jail on, read-only on initially, remote access off. Read-only blocks shell and all mutating tools. Time-limited elevation; cancel tracked jobs when disabling shell, disclose that detached processes may survive. Never allow AI to alter permissions or approve its own request.
- AI tool descriptions and MCP initialize instructions describe actual permissions; a status tool returns current policy. Server enforcement is authoritative; chat warnings are best effort. Risky operation approval must be local and bound to exact arguments, client, workspace and expiry; no generic approval replay.
- Protect app/config/token directories from file tools even when broad project access is selected; block symlink escapes and special files. Native shell can still access anything the OS account can access. No inherited secrets where avoidable.
- No runtime calls to upstream author services. Optional upstream check queries public GitHub commit metadata only, ideally in CI rather than the local app. Notify owner of new upstream commits and compare diffs; never download or execute upstream code automatically. Maintain MIT attribution.

## Reliability
Existing server reports online even when tunnel exits; this is a concrete defect but not proven to be the cause of observed 2–3 hour failure. Track local daemon, tunnel process, public HTTPS probe, OAuth refresh, MCP session lifecycle and recent successful client call independently. Probe public endpoint with a minimal non-sensitive health response, timeout and backoff. Keep fixed hostname and port; reconnect with bounded exponential backoff, distinguish sleep/offline from auth failures. Do not retry mutating calls blindly. Persist credentials securely, test reconnect and refresh concurrency, bound session and job maps. Show last check, last successful remote call, reason and repair action. Test sleep/wake and Wi-Fi transitions; cannot guarantee remote access when Mac is asleep.

## Interaction
Preserve current file/read/write/search/edit/git/job/fetch tools; add bounded pagination, binary metadata, atomic edits with expected hashes, structured results and resumable task status where protocol allows. Follow official MCP streamable HTTP and OAuth specifications; legacy SSE only if compatibility requires it and revocation is enforced. Use client capability negotiation and avoid assuming proprietary ChatGPT Work/Codex internals.

## Release gates
1. Unauthenticated requests cannot issue usable credentials or read files. Remote dashboard access fails. Revocation invalidates every transport and session. Test path escapes, unsafe git configuration, outbound DNS pinning, read-only and shell interactions.
2. All original tools have parity tests; actual ChatGPT, Claude and Gemini integrations tested independently (including refresh/restart).
3. 8-hour and 24-hour soak tests with idle periods, sleep/wake, lost network and tunnel restart; stable endpoint recovery without re-registration when supported.
4. Measure idle RSS, CPU, installation size and startup time on macOS, separately for app and tunnel. Initial targets: core idle RSS < 80 MB, idle CPU < 1%, dashboard ready < 3 seconds; targets not measured.
5. Native menu bar, explicit shutdown, clear access mode guidance, signed/notarized distribution where feasible; no release until security and compatibility gates pass.

## User-facing modes
- Off: remote access disabled.
- Review: read-only, folder locked, shell disabled.
- Edit: write access inside selected workspace, shell disabled.
- Run: native commands enabled for a limited interval after local confirmation; explicitly warns about whole-account access.
- Advanced: wider file scope only after local confirmation; never implies shell sandboxing.
