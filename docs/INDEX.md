# Networking Documentation Index

This directory documents Wicofe userspace networking configuration, services, and diagnostics.

## Scope
- Interface, address, route, and DNS configuration.
- Connection management and network policy.
- Userspace network daemons, IPC, and administrative tools.
- Protocol utilities, diagnostics, and integration tests.

## Ownership
Kernel packet I/O, network drivers, and kernel networking mechanisms belong to [Wicofe-Kernel](https://github.com/Waizusora/Wicofe-Kernel), unless a future architecture decision assigns a specific subsystem elsewhere. This repository owns userspace policy and service integration; it must not duplicate the kernel network stack.

## Security
Treat packets, peer metadata, and configuration input as untrusted. Document privileges, firewall/policy boundaries, secret storage, and protocol exposure.

- [Central documentation index](https://github.com/Waizusora/Wicofe-Docs/blob/main/INDEX.md)
- [Repository documentation map](https://github.com/Waizusora/Wicofe-Docs/blob/main/REPOSITORY_DOCUMENTATION_MAP.md)
