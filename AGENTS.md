# Repository Purpose

This repository is a minimal mirror for generated IPTV playlist files from:

https://guovin.github.io/iptv-api/

All mirrored outputs live under `lyrics/`:

- `lyrics/result.m3u`
- `lyrics/result.txt`
- `lyrics/ipv4.m3u`
- `lyrics/ipv4.txt`
- `lyrics/epg.gz`

It does not generate, discover, validate, proxy, or host IPTV streams.

EPG note: Guovin no longer publishes `epg.gz` (its EPG sources are empty),
so the workflow mirrors the EPG programme guide from
`https://epg.zsdc.eu.org/t.xml.gz` (suzukua/epg) instead, then adapts it
(channel ids and display-names, exact + normalized matching only) to the
naming used by the mirrored M3U/TXT playlists.
See `EPG_SOURCE_URL` and the "Adapt EPG channels to playlist naming" step
in `.github/workflows/update.yml`.

# Architecture

All update logic lives in:

`.github/workflows/update.yml`

Workflow name (as shown in GitHub Actions runs): `Update Lyrics`.

# Rules

- Keep the repository minimal.
- Do not add Docker, servers, databases, web UIs, or application frameworks.
- Do not proxy IPTV stream traffic.
- Do not manually edit mirrored output files.
- Changes to mirrored files must come from the GitHub Actions workflow.
- Treat the five mirrored files as one atomic update batch.
- A failed download or validation must preserve the previous known-good batch.
- Only rewrite the upstream EPG URL in M3U files (pointing at
  `lyrics/epg.gz`); never rewrite IPTV stream URLs.
- EPG channel id/name adaptation happens only on the EPG side; playlist
  files are never rewritten beyond the EPG URL.
- Do not hardcode the GitHub username when `${GITHUB_REPOSITORY}` can be used.
- Do not introduce PATs or other persistent credentials.
- Prefer small, auditable workflow changes.
