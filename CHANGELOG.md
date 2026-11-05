# Changelog

## [1.0.1] — 2026-10-22

### Added
- mdev vGPU passthrough support (NVIDIA vGPU with mediated devices)
- BAR alignment test in tier1 test catalog
- System-level error detection: host dmesg, VM dmesg, and libvirt log analysis
- GitLab CI support
- Pre-commit format standards

### Changed
- Replace paramiko with subprocess SSH for remote command execution

### Fixed
- Fix "destory" typo
- Fix pattern specificity and add traceability for error detection
- Fix device name validation for tower systems
- Add mdev type and vgpu type checks

## [1.0.0] — 2026-09-10

Initial release
