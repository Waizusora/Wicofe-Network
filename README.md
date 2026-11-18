# Wicofe-Network

Userspace networking configuration, services, and diagnostic tools for Wicofe OS.

## Scope

- Network configuration and connection management.
- Userspace network services and IPC.
- Address, route, DNS, and interface administration tools.
- Diagnostics and network integration tests.
- Optional protocol utilities and policy components.

## Boundaries

Network device drivers, packet I/O primitives, and kernel networking mechanisms belong in Wicofe-Kernel unless a later architecture decision assigns a specific component elsewhere. This repository owns userspace integration and policy, not a duplicate kernel network stack.

## Security

Treat network input as untrusted. Document privilege requirements, configuration permissions, secret handling, and protocol exposure.

## Status

Initial repository baseline; supported protocols and service architecture remain to be specified.

## License

Original project code targets MPL-2.0; protocol libraries and dependencies require individual review.

## Documentation

See [docs/INDEX.md](docs/INDEX.md) for this repository's technical documentation. The [central repository documentation map](https://github.com/Waizusora/Wicofe-Docs/blob/main/REPOSITORY_DOCUMENTATION_MAP.md) identifies canonical ownership and cross-repository boundaries.

## Wicofe-wide principles

Wicofe is a Unix-like, POSIX-compatible, non-GNU alternative. Native components must use Wicofe interfaces and must not require Linux libc or GNU compatibility layers. Prefer permissive licenses; avoid GPL/AGPL, and consider LGPL only when no suitable permissive alternative exists. Reuse or fork upstream software when implementing it from scratch would be disproportionately difficult. See the [canonical project principles](https://github.com/Waizusora/Wicofe-Docs/blob/main/PROJECT_PRINCIPLES.md).
