# PHP Windows Builds

PGO-optimized PHP builds for Windows x64 NTS, compiled locally on an AMD Zen 5 server.

Built because [windows.php.net](https://windows.php.net) has been frozen since December 17, 2025 — no official Windows builds available for recent PHP releases.

## Downloads

Check [Releases](https://github.com/lakeworks/php-builds/releases) for the latest builds.

## Build Specifications

| Spec | Value |
|------|-------|
| **Architecture** | x64 (AMD64) |
| **Thread Safety** | NTS (Non-Thread Safe) — for IIS FastCGI |
| **Optimization** | PGO (Profile-Guided Optimization) |
| **CPU Target** | AVX-512 (Zen 5) — also uses AVX2, SSE4.2 |
| **Compiler Flags** | `/Ox /GL /Qspectre /guard:cf /arch:AVX512` |
| **Build Config** | `--enable-snapshot-build` (all extensions enabled) |

## PHP Versions

| Branch | VS Toolset | Compiler | Windows SDK |
|--------|-----------|----------|-------------|
| **8.3.x** | vs16 (VS 2019, MSVC 14.29) | Visual C++ 2019 | 10.0.22621.0 |
| **8.4.x** | vs17 (VS 2022, MSVC 14.4x) | Visual C++ 2022 | 10.0.26100.0 |
| **8.5.x** | vs17 (VS 2022, MSVC 14.4x) | Visual C++ 2022 | 10.0.26100.0 |

## Extensions Included (40 shared DLLs)

All extensions enabled via `--enable-snapshot-build`:

bz2, com_dotnet, curl, dba, dl_test, enchant, exif, ffi, fileinfo, ftp,
gd, gettext, gmp, imap, intl, ldap, mbstring, mysqli, oci8_19, odbc,
opcache, openssl, pdo_firebird, pdo_mysql, pdo_oci, pdo_odbc, pdo_pgsql,
pdo_sqlite, pgsql, shmop, snmp, soap, sockets, sodium, sqlite3, sysvshm,
tidy, xsl, zend_test, zip

## Dependency Libraries

Dependencies fetched from `downloads.php.net/~windows/php-sdk/deps/` (staging branch — actively maintained even while windows.php.net is frozen).

### Stock Dependencies (from PHP SDK server)

| Library | Version | Notes |
|---------|---------|-------|
| OpenSSL | 3.0.19 | Uses AES-NI/AVX2 via runtime detection |
| cURL | 8.18.0 (package) | Static lib reports 8.7.0-DEV (upstream packaging bug) |
| ICU | 72.1 | Unicode 15.0 |
| libxml2 | 2.10.4 | |
| zlib | 1.2.12 | |
| PCRE2 | 10.42 | |
| libpng | 1.6.34 | |
| libjpeg-turbo | 2.1.0 | |
| FreeType | 2.11.1 | |
| libssh2 | 1.10.0 | |
| libwebp | 1.3.2 | |
| sqlite3 | 3.40.0 | |
| libzip | 1.7.1 | |
| oniguruma | 6.9.8 | |
| nghttp2 | 1.59.0 | |
| libsodium | 1.0.18 | |
| libpq | 14.21 | PostgreSQL client |
| BZip2 | 1.0.8 | |
| iconv | 1.17 | |

### Optimized Dependencies (rebuilt from source with AVX-512)

The following libraries are rebuilt from source with `/arch:AVX512` targeting the server's AMD Zen 5 CPU. These replace the generic x64 binaries from the PHP SDK server.

| Library | Stock → Rebuilt | Key Improvement |
|---------|----------------|-----------------|
| zlib → **zlib-ng** | 1.2.12 → 2.2.x | 2-5x faster gzip (AVX-512 VBMI2 deflate) |
| **libjpeg-turbo** | 2.1.0 → 3.1.x | AVX-512 JPEG encode/decode for product images |
| **PCRE2** | 10.42 → 10.45 | AVX2 JIT for regex matching |
| **libcurl** | 8.7.0-DEV → 8.12.1 | Fixed static lib version, HTTP/2 performance |
| **libpng** | 1.6.34 → 1.6.47 | SSE2/AVX2 PNG filters |
| **libwebp** | 1.3.2 → 1.5.0 | SIMD WebP encode/decode |
| **libxml2** | 2.10.4 → 2.13.6 | Security fixes + performance |
| **sqlite3** | 3.40.0 → 3.48.0 | Latest stable |
| **FreeType** | 2.11.1 → 2.13.3 | Minor improvements |

## Build Environment

| Component | Details |
|-----------|---------|
| **Host** | OVH Rise-S-2 dedicated server |
| **CPU** | AMD Ryzen 7 9700X (Zen 5, 8C/16T, 5.5 GHz boost) |
| **RAM** | 64 GB DDR5 5200 MHz |
| **OS** | Windows Server 2025 Standard (10.0.26100) |
| **Virtualization** | Proxmox 8.4 / KVM (`cpu: host,flags=+aes`) |
| **CPU Features** | SSE4.2, AVX, AVX2, AVX-512, AES-NI |

## Build Toolchain

Built using a [fork of php-windows-builder](https://github.com/lakeworks/php-windows-builder) with fixes for hardened Windows Server environments:

- WebClient downloads (bypass WPAD proxy hangs)
- vswhere `-all` (find older toolsets in VS 2022)
- TLS 1.2 only (hardened server compatibility)
- Windows SDK 22621 compatibility (SDK 26100 UCRT CFG linker fix for v142)
- Git worktree preservation (robocopy instead of Move-Item)
- Temporary firewall rules for SDK php.exe (try/finally cleanup)

## Verification

Each release includes a `sha256sum.txt` file. Verify with:

```powershell
$hash = (Get-FileHash php-8.3.30-nts-Win32-vs16-x64.zip -Algorithm SHA256).Hash.ToLower()
$expected = (Get-Content sha256sum.txt).Split(' ')[0]
if ($hash -eq $expected) { "OK" } else { "MISMATCH" }
```

## Compatibility Note

These builds use `/arch:AVX512` and are optimized for AMD Zen 5 CPUs. They require a processor with AVX-512 support. They will **not run** on older CPUs without AVX-512.

For generic x64 builds (SSE2 baseline), use the stock builds from [windows.php.net](https://windows.php.net) or [shivammathur/php-builder-windows](https://github.com/shivammathur/php-builder-windows).

## License

PHP is licensed under the [PHP License](https://www.php.net/license/). The build scripts are licensed under [MIT](https://github.com/lakeworks/php-windows-builder/blob/master/LICENSE).
