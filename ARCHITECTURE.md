# Architecture: curl

## Purpose

The curl C library — a client-side URL transfer library supporting HTTP, HTTPS, FTP, SFTP, SMTP, and dozens of other protocols. This is the upstream C source repository (curl/curl), not a PHP wrapper. It is included in this datasmith collection for reference purposes.

## Directory Structure

```
lib/                     — Core library source (C): protocol handlers, SSL backends, connection management
src/                     — curl command-line tool source (C)
include/curl/            — Public C headers: curl.h (API), multi.h (multi-handle), easy.h
tests/                   — Test infrastructure: Python-based test server + C test programs
docs/                    — Documentation: man pages, FAQ, internals guides
CMakeLists.txt           — CMake build system
configure.ac             — Autotools build system
```

## Key Design Decisions

- **Easy vs Multi interface** — `curl_easy_*` provides synchronous single-transfer API; `curl_multi_*` provides an event-driven multiplexed API for concurrent transfers without threads.
- **Pluggable SSL backends** — OpenSSL, GnuTLS, mbedTLS, wolfSSL, Schannel, and others are selectable at build time; a thin abstraction layer (`vtls/`) dispatches to the active backend.
- **Protocol handler abstraction** — each protocol (HTTP, FTP, SMTP, etc.) registers a handler struct with connect/do/done function pointers; the transfer engine calls them generically.

Note: This is a C codebase. It is not PHP and contains no PHP source files.

## Extension Points

- Build with a custom SSL backend by selecting it at configure/cmake time.
- Link against libcurl from PHP via the `curl` extension (`ext/curl`) in the PHP interpreter.

## Dependency Flow

```
curl_easy_perform(handle)
  └── transfer engine (lib/transfer.c)
        ├── protocol handler (lib/http.c, lib/ftp.c, ...)
        ├── SSL backend (lib/vtls/openssl.c, ...)
        └── socket I/O → network → response data
```
