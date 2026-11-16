# Wicofe Userspace Networking Architecture

- **Version:** 0.1
- **Status:** Draft architecture
- **Repository:** Wicofe-Network
- **Targets:** x86-64 and AArch64
- **License:** MPL-2.0 for original project code

## 1. Purpose and ownership

Wicofe-Network owns userspace network configuration, connection management, network policy integration, diagnostics, and network services. It does not own the kernel packet-processing stack or hardware drivers.

Ownership boundaries:
- **Wicofe-Kernel:** network device model, packet I/O primitives, socket/syscall implementation, protocol mechanisms explicitly assigned to kernel space, and driver-facing interfaces.
- **Wicofe-Drivers:** hardware-specific Ethernet, Wi-Fi, and other network-device drivers.
- **Wicofe-ABI:** userspace-visible socket, ioctl/control, event, and error contracts.
- **Wicofe-Network:** configuration tools, network manager/service, DNS client/resolver integration, connection policy, diagnostics, and userspace protocol utilities.
- **Wicofe-Services:** common service supervision, logging, and system-wide service lifecycle where shared.

This repository must consume published kernel and ABI contracts. It must not duplicate kernel packet I/O or invent private syscall numbers.

## 2. Layered architecture

The intended architecture is:

1. **Device and link layer (kernel):** hardware drivers and link-state reporting.
2. **Network stack (kernel):** packet buffers, routing hooks, sockets, and protocol implementation according to kernel scope.
3. **Control interface (ABI):** versioned operations and events for interfaces, addresses, routes, neighbors, and statistics.
4. **Network service (userspace):** policy, configuration reconciliation, connection state, and orchestration.
5. **Resolver and supporting services:** DNS configuration, name resolution, time/network dependencies, and optional discovery.
6. **Administrative tools and UI integration:** inspection, configuration, diagnostics, and desktop status integration.

Exact kernel/userspace placement of individual protocols remains a system architecture decision. Do not assume every protocol must run in userspace or kernel space.

## 3. Core userspace components

### 3.1 Network manager

Maintains desired and observed network state. It should reconcile:
- Interface administrative state.
- Link state and negotiated capabilities.
- IPv4/IPv6 address configuration.
- Routes and route metrics.
- DNS resolver configuration.
- Wireless association state where supported.
- Policy for metered, roaming, and untrusted networks where applicable.

The manager must distinguish requested state from confirmed kernel/device state. A command being accepted does not mean a connection is established.

### 3.2 Configuration model

Configuration should be structured and versioned rather than arbitrary shell snippets. It should support:
- Static and dynamic addressing.
- IPv4 and IPv6.
- Multiple interfaces and route priorities.
- DNS servers and search domains.
- Wireless profiles where a supported supplicant exists.
- Per-profile permissions and secret references.
- Automatic connection policy and explicit user overrides.

Secrets must not be stored in plaintext configuration or diagnostics. Use a dedicated credential mechanism once the system secret-storage contract exists.

### 3.3 Resolver integration

The resolver component manages configured DNS sources and exposes a documented local resolution interface. It must define:
- Upstream server selection and failover.
- Search-domain behavior.
- Cache limits and expiry.
- DNS over UDP/TCP behavior and supported extensions.
- IPv4/IPv6 preference and fallback.
- Negative caching.
- Split DNS policy if supported.
- Behavior when the network service is unavailable.

Do not claim encrypted DNS support until certificate validation, trust roots, bootstrap resolution, and privacy policy are implemented and tested.

### 3.4 Wireless management

Wireless support may require a dedicated supplicant and backend integration. It must define:
- Supported hardware and driver interfaces.
- Supported authentication and encryption suites.
- Credential storage and access control.
- Scan and association lifecycle.
- Roaming and reconnect policy.
- Regulatory-domain handling.
- Behavior on driver reset and device removal.

Do not invent or downgrade security suites silently. Unsupported or insecure configurations must be reported clearly.

### 3.5 Diagnostics

Diagnostics should report observed interface state, addresses, routes, resolver state, link counters, and recent service errors. Sensitive network identifiers and credentials must be redacted according to policy.

## 4. Control plane and event model

The network manager requires a versioned control interface to query and modify network state. The final interface belongs to Wicofe-ABI and kernel specifications.

Requirements:
- Enumerate interfaces using stable kernel-issued identifiers where available.
- Subscribe to link, address, route, and device lifecycle events.
- Detect event loss and support full state resynchronization.
- Make configuration operations idempotent where practical.
- Validate interface identity again before applying delayed operations.
- Return structured errors for permission, unsupported operation, stale object, and device failure.
- Separate read-only observation privileges from configuration privileges.

A userspace manager must not assume that event delivery is lossless. It should periodically or on event-gap indication reconcile its cached state against an authoritative snapshot.

## 5. Connection lifecycle

A managed connection should have explicit states such as:
- Disconnected
- Preparing
- Associating
- Authenticating
- Configuring
- Online
- Degraded
- Disconnecting
- Failed

Not every transport uses every state. State transitions must be driven by observed events and bounded operation results, not by optimistic UI assumptions.

Reconnect behavior must use bounded backoff and avoid tight retry loops. User-requested disconnect must suppress automatic reconnect according to documented policy.

## 6. Security model

- Treat packets, DHCP options, DNS responses, wireless scan results, and peer metadata as untrusted.
- Require authorization for interface, route, address, and resolver changes.
- Keep wireless credentials and other secrets out of logs and crash reports.
- Validate configuration files, ownership, permissions, and schema versions.
- Avoid exposing privileged control sockets to untrusted users.
- Prevent DNS or route updates from silently overriding protected system policy.
- Apply least privilege to daemons and helper processes.
- Document firewall integration and default inbound exposure before enabling services.

The exact capability/authorization model is owned by Wicofe's security and service architecture.

## 7. Failure and recovery behavior

The manager must handle:
- Interface disappearance or renaming.
- Driver reset and link flapping.
- DHCP timeout or invalid lease.
- Duplicate address detection failure.
- Route conflict and unreachable gateway.
- DNS timeout or malformed response.
- Wireless authentication failure.
- Service restart while configuration remains active.
- Kernel control API version mismatch.

On restart, the manager should inspect actual system state before applying desired configuration. It must not blindly duplicate addresses or routes.

## 8. Logging and observability

Logs should include operation IDs, interface identity, state transitions, failure stage, and structured error codes. Do not log passphrases, private keys, authentication tokens, or full packet payloads by default.

Metrics may include link transitions, packet/error counters exposed by the kernel, address acquisition duration, resolver failures, and reconnect attempts. Metrics must not be represented as proof of end-to-end connectivity.

## 9. Testing strategy

- Configuration schema validation and version migration.
- Interface hotplug and identifier reuse.
- Event loss followed by state resynchronization.
- Static and dynamic IPv4/IPv6 configuration.
- Route conflict and metric selection.
- Resolver failover, malformed responses, cache expiry, and negative caching.
- Wireless authentication failure and credential redaction.
- Daemon restart and reconciliation.
- Privilege boundary and unauthorized mutation tests.
- Network loss during active operations.
- x86-64 and AArch64 build and integration coverage.

Use isolated virtual networks and emulated devices for automated tests. Physical wireless interoperability must be recorded separately from emulation results.

## 10. Repository layout

Suggested organization:

```text
src/
  manager/
  config/
  resolver/
  wireless/
  diagnostics/
  policy/
  ipc/
tests/
  config/
  virtual_network/
  resolver/
  security/
docs/
  INDEX.md
  NETWORKING_ARCHITECTURE_0.1.md
```

This is a target layout, not a claim that these components already exist.

## 11. Decisions still open

- Exact kernel network stack scope and protocol placement.
- Stable interface identity and event subscription ABI.
- Socket and network-control syscall contracts.
- Network manager IPC and privilege model.
- Initial DHCP client and IPv6 autoconfiguration strategy.
- DNS resolver implementation and encrypted DNS policy.
- Wireless supplicant choice and license review.
- Firewall subsystem and default policy.
- Initial supported wired and wireless hardware.

## 12. Completion criteria

The subsystem is not complete when an interface merely receives an address. Completion requires versioned control/event contracts, safe privilege separation, correct state reconciliation, resolver behavior, device hotplug handling, security tests, and reproducible virtual and physical interoperability evidence.

**This document is a draft architecture; it does not claim that a network manager, resolver, supplicant, or protocol service is implemented.**
