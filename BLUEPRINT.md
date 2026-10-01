# Radio Clash Repository Setup Instructions

## 1. Objective

Set up a standalone, public GitHub repository named:

```text
radio-clash
```

The repository exists solely to mirror pre-generated IPTV playlist outputs from `Guovin/iptv-api` on a fixed schedule.

This repository is **not** a fork of `Guovin/iptv-api`.

It must **not** run any of the upstream project's source discovery, validation, speed testing, filtering, playlist generation, or IPTV stream proxying logic.

Its only responsibilities are:

1. Download the already-generated upstream output files.
2. Validate them.
3. Download the EPG programme guide from `suzukua/epg`, validate it, and adapt its channel naming to the playlists.
4. Rewrite the EPG URL inside M3U files so the TV does not need direct access to GitHub Pages.
5. Commit updated copies into a normal Git repository, under `lyrics/`.
6. Expose stable GitHub Raw URLs that can be accessed through a GitHub Raw proxy.

The four playlists must be mirrored together:

```text
https://guovin.github.io/iptv-api/result.m3u
https://guovin.github.io/iptv-api/result.txt
https://guovin.github.io/iptv-api/ipv4.m3u
https://guovin.github.io/iptv-api/ipv4.txt
```

The EPG programme guide is mirrored from `suzukua/epg` instead:

```text
https://epg.zsdc.eu.org/t.xml.gz
```

Historical note: the original design mirrored `https://guovin.github.io/iptv-api/epg.gz`. Guovin stopped publishing that file (its upstream run logs report "EPG source count: 0"), so the EPG is now sourced from `suzukua/epg` (Cloudflare Pages, updated at least twice daily, valid gzip XMLTV). The M3U files still reference the old Guovin EPG URL in their `x-tvg-url` header, which is why that URL is rewritten.

The target repository already exists:

```text
https://github.com/Shawnkurt/radio-clash
```

Repository:

```text
Shawnkurt/radio-clash
```

The final implementation must:

1. Use the existing public repository `Shawnkurt/radio-clash`.
2. Run a GitHub Actions workflow every 24 hours.
3. Download and validate all five files as a single batch:
   - `result.m3u`
   - `result.txt`
   - `ipv4.m3u`
   - `ipv4.txt`
   - `epg.gz` (from `https://epg.zsdc.eu.org/t.xml.gz`)
4. Treat all five files as one atomic update batch:
   - all files must download successfully;
   - all files must pass validation;
   - only then may the repository files be replaced;
   - if any file fails, preserve the previous known-good batch.
5. Rewrite the upstream GitHub Pages EPG URL inside both M3U files to point at `lyrics/epg.gz` in this repository.
6. Adapt the EPG channel ids and display-names to the naming used by the mirrored playlists (exact + normalized matching only).
7. Commit only when mirrored content has actually changed.
8. Provide stable Raw URLs suitable for use through a GitHub Raw proxy.
9. Manually trigger the workflow once after setup and verify the first synchronization succeeds.

---

## 2. Final Data Flow

```text
Guovin GitHub Pages            suzukua/epg (Cloudflare Pages)
        │                              │
        ├── result.m3u                 └── t.xml.gz
        ├── result.txt
        ├── ipv4.m3u
        └── ipv4.txt
        │
        ▼
GitHub Actions (workflow name: "Update Lyrics")
Download + validate + rewrite EPG URL + adapt EPG naming
every 24 hours
        │
        ▼
radio-clash / main
        │
        ├── lyrics/result.m3u
        ├── lyrics/result.txt
        ├── lyrics/ipv4.m3u
        ├── lyrics/ipv4.txt
        └── lyrics/epg.gz
        │
        ▼
raw.githubusercontent.com
        │
        ▼
GitHub Raw proxy
        │
        ▼
TV IPTV player
```

Important:

- The GitHub proxy is only used to retrieve playlist and EPG files.
- Actual IPTV stream URLs contained inside M3U/TXT files must remain unchanged.
- The player should connect directly to those IPTV stream URLs after loading the playlist.

---

## 3. Execution Rules

The coding agent must follow these rules:

1. Prefer `gh` CLI for repository creation, authentication checks, workflow execution, and workflow status inspection.
2. Keep all operations idempotent where practical.
3. If the repository already exists, do not delete it and do not destroy unrelated configuration.
4. Do not fork `Guovin/iptv-api`.
5. Do not copy the upstream project source code.
6. Do not run IPTV discovery, validation, speed testing, source filtering, or playlist generation.
7. This repository is strictly a mirror of already-generated upstream outputs (plus the `suzukua/epg` programme guide).
8. Download files into a temporary directory first.
9. Never download directly over the live repository copies.
10. If any download fails, fail the entire batch.
11. If any validation fails, fail the entire batch.
12. On failure, preserve the previous known-good files.
13. Never commit HTML error pages, empty files, truncated files, or invalid gzip data.
14. Use the repository-provided `GITHUB_TOKEN`.
15. Do not create or commit PATs, cookies, passwords, or other credentials.
16. Do not hardcode the GitHub username inside the workflow.
17. Use `${GITHUB_REPOSITORY}` to determine the current repository dynamically.
18. After setup, manually trigger the workflow and fix any issues until the first run succeeds.
19. Do not create a README file.
20. Keep the repository minimal.

---

## 4. Repository Definition

The repository has already been created. Do not create, rename, delete, or replace it.

Repository URL:

```text
https://github.com/Shawnkurt/radio-clash
```

Repository:

```text
Shawnkurt/radio-clash
```

Repository name:

```text
radio-clash
```

Visibility:

```text
public
```

Default branch:

```text
main
```

Final repository structure:

```text
radio-clash/
├── .github/
│   └── workflows/
│       └── update.yml
├── .gitattributes
└── lyrics/
    ├── result.m3u
    ├── result.txt
    ├── ipv4.m3u
    ├── ipv4.txt
    └── epg.gz
```

The five mirrored files live under `lyrics/`.

They should be created and updated automatically by successful workflow runs.

No README file should be created.

---

## 5. Prerequisite Checks

Check GitHub CLI:

```bash
gh --version
gh auth status
```

If GitHub CLI is not authenticated, stop all GitHub write operations and tell the user to run:

```bash
gh auth login
```

Determine the current GitHub username:

```bash
GH_USER="$(gh api user --jq .login)"
echo "$GH_USER"
```

Check Git:

```bash
git --version
```

---

## 6. Clone and Verify the Existing Repository

The GitHub repository already exists:

```text
https://github.com/Shawnkurt/radio-clash
```

Do not run `gh repo create`.

If the repository is not already present locally, clone it:

```bash
git clone https://github.com/Shawnkurt/radio-clash.git
cd radio-clash
```

If the current working directory is already the local checkout, do not clone another copy.

Verify the remote and branch:

```bash
git remote -v
git branch --show-current
gh repo view Shawnkurt/radio-clash
```

The expected origin is:

```text
https://github.com/Shawnkurt/radio-clash.git
```

The expected repository is:

```text
Shawnkurt/radio-clash
```

Confirm that:

- the repository is public;
- the default branch is `main`;
- the authenticated GitHub account has push/admin access;
- the local `origin` points to `Shawnkurt/radio-clash`;
- the working branch is `main`.

If the repository is currently empty, that is expected. Initialize its contents by creating only the files required by this instruction.

Do not create a README.

## 7. GitHub Raw Proxy Configuration

Use the following proxy prefix by default:

```text
https://ghfast.top/
```

Define it once at the workflow level:

```yaml
env:
  GH_PROXY_PREFIX: "https://ghfast.top/"
```

This allows the proxy to be replaced later by changing a single value.

For a repository:

```text
Shawnkurt/radio-clash
```

the TV-facing URLs will be:

### Full M3U

```text
https://ghfast.top/https://raw.githubusercontent.com/Shawnkurt/radio-clash/main/lyrics/result.m3u
```

### Full TXT

```text
https://ghfast.top/https://raw.githubusercontent.com/Shawnkurt/radio-clash/main/lyrics/result.txt
```

### IPv4 M3U

```text
https://ghfast.top/https://raw.githubusercontent.com/Shawnkurt/radio-clash/main/lyrics/ipv4.m3u
```

### IPv4 TXT

```text
https://ghfast.top/https://raw.githubusercontent.com/Shawnkurt/radio-clash/main/lyrics/ipv4.txt
```

### EPG

```text
https://ghfast.top/https://raw.githubusercontent.com/Shawnkurt/radio-clash/main/lyrics/epg.gz
```

Do not hardcode `username` inside the workflow.

Use:

```text
${GITHUB_REPOSITORY}
```

instead.

---

## 8. Create `.gitattributes`

Create:

```text
.gitattributes
```

with:

```gitattributes
*.gz binary
```

---

## 9. Create the GitHub Actions Workflow

Create:

```text
.github/workflows/update.yml
```

with the following content:

```yaml
name: Update Lyrics

on:
  workflow_dispatch:
  schedule:
    # GitHub Actions cron is UTC.
    # Run once every 24 hours and avoid exact-hour congestion.
    - cron: "17 0 * * *"

permissions:
  contents: write

concurrency:
  group: update-lyrics
  cancel-in-progress: false

env:
  UPSTREAM_BASE: "https://guovin.github.io/iptv-api"
  # Guovin no longer publishes epg.gz (EPG sources count is 0), so the EPG
  # programme guide is mirrored from suzukua/epg instead.
  EPG_SOURCE_URL: "https://epg.zsdc.eu.org/t.xml.gz"
  GH_PROXY_PREFIX: "https://ghfast.top/"

jobs:
  update:
    runs-on: ubuntu-latest
    timeout-minutes: 15

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Download upstream batch
        shell: bash
        run: |
          set -euo pipefail

          rm -rf .tmp
          mkdir -p .tmp

          download() {
            local name="$1"

            echo "Downloading ${name} ..."

            curl \
              --fail \
              --location \
              --silent \
              --show-error \
              --retry 5 \
              --retry-delay 5 \
              --retry-all-errors \
              --connect-timeout 20 \
              --max-time 180 \
              "${UPSTREAM_BASE}/${name}" \
              --output ".tmp/${name}"
          }

          download result.m3u
          download result.txt
          download ipv4.m3u
          download ipv4.txt

          echo "Downloading epg.gz (from ${EPG_SOURCE_URL}) ..."
          curl \
            --fail \
            --location \
            --silent \
            --show-error \
            --retry 5 \
            --retry-delay 5 \
            --retry-all-errors \
            --connect-timeout 20 \
            --max-time 180 \
            "${EPG_SOURCE_URL}" \
            --output ".tmp/epg.gz"

      - name: Validate downloaded batch
        shell: bash
        run: |
          set -euo pipefail

          reject_html_or_error_page() {
            local file="$1"

            if grep -aEqi \
              '<html|<!doctype html|rate limit|access denied|forbidden|bad gateway|service unavailable' \
              "$file"; then
              echo "${file} appears to contain an HTML/error response."
              exit 1
            fi
          }

          validate_m3u() {
            local file="$1"

            test -s "$file"

            # Reject unexpectedly tiny placeholder/error files.
            test "$(wc -c < "$file")" -gt 1024

            # Basic M3U structure.
            head -n 1 "$file" | grep -q '^#EXTM3U'
            grep -q '^#EXTINF:' "$file"

            reject_html_or_error_page "$file"
          }

          validate_txt() {
            local file="$1"

            test -s "$file"
            test "$(wc -c < "$file")" -gt 1024

            # Playlist should contain at least one network stream URL.
            grep -aEq 'https?://|rtmp://|rtsp://|udp://|rtp://' "$file"

            reject_html_or_error_page "$file"
          }

          validate_m3u .tmp/result.m3u
          validate_m3u .tmp/ipv4.m3u

          validate_txt .tmp/result.txt
          validate_txt .tmp/ipv4.txt

          test -s .tmp/epg.gz

          # Must be a valid gzip file.
          gzip -t .tmp/epg.gz

          # Decompress once, then check the XMLTV content.
          # (Do not pipe zcat into grep -q: grep -q exits early and zcat
          # would fail with SIGPIPE under pipefail.)
          zcat .tmp/epg.gz > .tmp/epg.xml

          # Reuse the same structural checks before and after adaptation.
          cat > .tmp/validate_batch.py <<'PY'
          import gzip
          import os
          import re
          import sys
          import xml.etree.ElementTree as ET
          from pathlib import Path

          rewritten = "--rewritten" in sys.argv
          mirror_epg = (
              os.environ["GH_PROXY_PREFIX"]
              + "https://raw.githubusercontent.com/"
              + os.environ["GITHUB_REPOSITORY"]
              + "/main/lyrics/epg.gz"
          )
          stream_url = re.compile(r"^(?:https?|rtmp|rtsp|udp|rtp)://\S+$")
          epg_attribute = re.compile(r'(?:x-tvg-url|url-tvg)="([^"]*)"')

          for name in ("result.m3u", "ipv4.m3u"):
              path = Path(".tmp") / name
              lines = path.read_text(encoding="utf-8-sig").splitlines()
              if not lines or not re.match(r"^#EXTM3U(?:\s|$)", lines[0]):
                  raise SystemExit(f"{path}: invalid M3U header")
              pending = False
              entries = 0
              for line in lines[1:]:
                  line = line.strip()
                  if line.startswith("#EXTINF:"):
                      if pending:
                          raise SystemExit(f"{path}: channel has no stream URL")
                      pending = True
                  elif line and not line.startswith("#"):
                      if not pending or not stream_url.fullmatch(line):
                          raise SystemExit(f"{path}: invalid or unpaired stream URL")
                      pending = False
                      entries += 1
              if pending or not entries:
                  raise SystemExit(f"{path}: missing stream URL or empty playlist")
              if rewritten:
                  urls = epg_attribute.findall(lines[0])
                  if not urls or any(url != mirror_epg for url in urls):
                      raise SystemExit(f"{path}: incorrect mirror EPG URL")

          with gzip.open(".tmp/epg.gz", "rb") as handle:
              root = ET.parse(handle).getroot()
          channels = root.findall("channel")
          programmes = root.findall("programme")
          if root.tag != "tv" or not channels or not programmes:
              raise SystemExit("EPG must have a tv root, channels and programmes")
          ids = [channel.get("id") for channel in channels]
          if any(not channel_id or not channel_id.strip() for channel_id in ids):
              raise SystemExit("EPG contains an empty channel id")
          if rewritten and len(ids) != len(set(ids)):
              raise SystemExit("Adapted EPG contains duplicate channel ids")
          known_ids = set(ids)
          if any(p.get("channel") not in known_ids for p in programmes):
              raise SystemExit("EPG programme references an unknown channel")
          print("Playlist structure and XMLTV references validated")
          PY

          python3 .tmp/validate_batch.py

      - name: Rewrite EPG URLs in M3U files
        shell: bash
        run: |
          set -euo pipefail

          export MIRROR_EPG_URL="${GH_PROXY_PREFIX}https://raw.githubusercontent.com/${GITHUB_REPOSITORY}/main/lyrics/epg.gz"

          python3 <<'PY'
          import os
          import re
          from pathlib import Path

          mirror_epg = os.environ["MIRROR_EPG_URL"]

          targets = [
              Path(".tmp/result.m3u"),
              Path(".tmp/ipv4.m3u"),
          ]

          for path in targets:
              # Only change EPG attributes on the header. Preserve every
              # subsequent byte, including stream URLs and line endings.
              text = path.read_bytes().decode("utf-8-sig")
              lines = text.splitlines(keepends=True)
              header = lines[0].rstrip("\r\n")
              ending = lines[0][len(header):]
              attribute = re.compile(r'(?:x-tvg-url|url-tvg)="[^"]*"')
              if attribute.search(header):
                  header = attribute.sub(
                      lambda match: match.group(0).split("=", 1)[0]
                      + f'="{mirror_epg}"', header
                  )
              else:
                  header += f' x-tvg-url="{mirror_epg}"'
              lines[0] = header + ending
              path.write_bytes("".join(lines).encode("utf-8"))
              print(f"{path}: EPG header set to {mirror_epg}")
          PY

      - name: Adapt EPG channels to playlist naming
        shell: bash
        run: |
          set -euo pipefail

          # Rewrite EPG channel ids and display-names so they match the
          # naming used by result.m3u / ipv4.m3u (tvg-id) and result.txt /
          # ipv4.txt (channel name). Only exact and normalized
          # (case/whitespace/hyphen-insensitive) matches are used.
          # The playlist files themselves are never modified here.

          python3 <<'PY'
          import gzip
          import re
          import xml.etree.ElementTree as ET
          from collections import defaultdict
          from pathlib import Path

          NORM_STRIP = re.compile(r"[\s\-_·・]")

          def norm(value):
              return NORM_STRIP.sub("", value or "").upper()

          # ---- Collect naming used by the mirrored playlists ----

          tvg_ids = set()
          names = set()
          tvg_id_re = re.compile(r'tvg-id="([^"]*)"')
          tvg_name_re = re.compile(r'tvg-name="([^"]*)"')

          for path in (".tmp/result.m3u", ".tmp/ipv4.m3u"):
              for line in Path(path).read_text(encoding="utf-8").splitlines():
                  if not line.startswith("#EXTINF"):
                      continue
                  match = tvg_id_re.search(line)
                  if match and match.group(1):
                      tvg_ids.add(match.group(1))
                  match = tvg_name_re.search(line)
                  if match and match.group(1):
                      names.add(match.group(1))
                  display = line.rsplit(",", 1)[-1].strip()
                  if display:
                      names.add(display)

          for path in (".tmp/result.txt", ".tmp/ipv4.txt"):
              for line in Path(path).read_text(encoding="utf-8").splitlines():
                  line = line.strip()
                  if not line or "#genre#" in line:
                      continue
                  channel_name = line.split(",", 1)[0].strip()
                  if channel_name:
                      names.add(channel_name)

          norm_to_tvg_id = defaultdict(set)
          for tid in tvg_ids:
              norm_to_tvg_id[norm(tid)].add(tid)
          norm_to_names = defaultdict(set)
          for name in names:
              norm_to_names[norm(name)].add(name)

          # ---- Adapt the EPG ----

          tree = ET.parse(".tmp/epg.xml")
          root = tree.getroot()

          id_map = {}
          merged_display = {}
          exact = 0
          renamed = 0
          aliases_added = 0

          for channel in root.findall("channel"):
              old_id = channel.get("id") or ""
              if old_id in tvg_ids:
                  final_id = old_id
                  exact += 1
              else:
                  candidates = norm_to_tvg_id.get(norm(old_id))
                  if candidates and len(candidates) == 1:
                      final_id = next(iter(candidates))
                      renamed += 1
                  else:
                      final_id = old_id
              id_map[old_id] = final_id

              display_list = merged_display.setdefault(final_id, [])
              for text in [e.text for e in channel.findall("display-name")]:
                  if text and text not in display_list:
                      display_list.append(text)
              for alias in sorted(norm_to_names.get(norm(old_id), ())):
                  if alias not in display_list:
                      display_list.append(alias)
                      aliases_added += 1

          # Rewrite kept channel elements, merge duplicates.
          seen = set()
          merged = 0
          for channel in list(root.findall("channel")):
              old_id = channel.get("id") or ""
              final_id = id_map[old_id]
              if final_id in seen:
                  root.remove(channel)
                  merged += 1
                  continue
              seen.add(final_id)
              channel.set("id", final_id)
              for element in channel.findall("display-name"):
                  channel.remove(element)
              for text in merged_display[final_id]:
                  element = ET.SubElement(channel, "display-name")
                  element.set("lang", "zh")
                  element.text = text

          programmes_renamed = 0
          for programme in root.findall("programme"):
              old_channel = programme.get("channel")
              if old_channel in id_map and id_map[old_channel] != old_channel:
                  programme.set("channel", id_map[old_channel])
                  programmes_renamed += 1

          total = len(root.findall("channel"))
          matched = sum(
              1 for c in root.findall("channel") if c.get("id") in tvg_ids
          )
          print(f"EPG adaptation: {total} channels (after merge)")
          print(f"  exact id match:      {exact}")
          print(f"  normalized rename:   {renamed}")
          print(f"  duplicates merged:   {merged}")
          print(f"  alias names added:   {aliases_added}")
          print(f"  programmes remapped: {programmes_renamed}")
          print(
              f"  channels matching playlist tvg-id: "
              f"{matched}/{total}"
          )

          # mtime=0 keeps the gzip output deterministic so unchanged
          # EPG content does not produce new commits.
          xml_bytes = ET.tostring(
              root, encoding="utf-8", xml_declaration=True
          )
          Path(".tmp/epg.xml").write_bytes(xml_bytes)
          Path(".tmp/epg.gz").write_bytes(
              gzip.compress(xml_bytes, 9, mtime=0)
          )

          # Round-trip check: re-parse and verify the gzip payload.
          ET.parse(".tmp/epg.xml")
          with gzip.open(".tmp/epg.gz", "rb") as handle:
              assert handle.read() == xml_bytes
          PY

      - name: Validate rewritten batch
        shell: bash
        run: |
          set -euo pipefail

          test -s .tmp/result.txt
          test -s .tmp/ipv4.txt
          gzip -t .tmp/epg.gz
          python3 .tmp/validate_batch.py --rewritten

      - name: Install validated batch
        shell: bash
        run: |
          set -euo pipefail

          # Only replace the live repository files after the complete batch
          # has downloaded and validated successfully.

          mkdir -p lyrics

          mv .tmp/result.m3u ./lyrics/result.m3u
          mv .tmp/result.txt ./lyrics/result.txt
          mv .tmp/ipv4.m3u ./lyrics/ipv4.m3u
          mv .tmp/ipv4.txt ./lyrics/ipv4.txt
          mv .tmp/epg.gz ./lyrics/epg.gz

      - name: Commit changes
        shell: bash
        run: |
          set -euo pipefail

          git config user.name "github-actions[bot]"
          git config user.email \
            "41898282+github-actions[bot]@users.noreply.github.com"

          git add \
            lyrics/result.m3u \
            lyrics/result.txt \
            lyrics/ipv4.m3u \
            lyrics/ipv4.txt \
            lyrics/epg.gz

          if git diff --cached --quiet; then
            echo "No upstream changes detected; nothing to commit."
            exit 0
          fi

          git commit -m "chore: update lyrics"
          git push
```

---

## 10. Atomic Update Requirement

This is one of the most important reliability requirements.

Treat these five files as one update batch:

```text
lyrics/result.m3u
lyrics/result.txt
lyrics/ipv4.m3u
lyrics/ipv4.txt
lyrics/epg.gz
```

The workflow must follow this sequence:

```text
Download all files into .tmp/
        ↓
Validate result.m3u
        ↓
Validate result.txt
        ↓
Validate ipv4.m3u
        ↓
Validate ipv4.txt
        ↓
Validate epg.gz (gzip + XMLTV content)
        ↓
Rewrite EPG URLs in both M3U files
        ↓
Adapt EPG channel naming to the playlists
        ↓
Validate again
        ↓
All checks pass
        ↓
Replace live repository files under lyrics/
        ↓
Create one commit
```

The following behavior is forbidden:

```text
result.m3u succeeds → replace live file
result.txt succeeds → replace live file
ipv4.m3u fails
```

That would leave the repository with files from different upstream batches.

Required behavior:

> All five files succeed together, or nothing is replaced.

---

## 11. EPG Handling

### Why the EPG must be mirrored

The upstream M3U contains a header similar to:

```m3u
#EXTM3U x-tvg-url="https://guovin.github.io/iptv-api/epg.gz"
```

If the TV network cannot reliably access GitHub Pages, the playlist may load through a Raw proxy while EPG still fails.

The URL is rewritten to:

```m3u
#EXTM3U x-tvg-url="https://ghfast.top/https://raw.githubusercontent.com/USER/radio-clash/main/lyrics/epg.gz"
```

Set only the `x-tvg-url` / `url-tvg` attributes on the M3U header to the mirror URL. Add `x-tvg-url` if missing, and replace old mirror URLs as well as upstream URLs. Preserve all subsequent playlist bytes.

Do not broadly rewrite other URLs.

In particular, never rewrite actual IPTV stream URLs.

Example:

```m3u
#EXTINF:-1,CCTV-1
http://1.2.3.4:8080/live/...
```

The stream URL:

```text
http://1.2.3.4:8080/live/...
```

must remain unchanged.

### EPG source

The EPG payload itself comes from:

```text
https://epg.zsdc.eu.org/t.xml.gz
```

(`suzukua/epg`, published via Cloudflare Pages, updated at least twice daily.)

### EPG channel-name adaptation

The source EPG names CCTV-style channels as `CCTV1`, while the playlists use
`tvg-id="CCTV-1"`. Players that match the EPG strictly by `tvg-id` (or, for
TXT playlists, by channel name) would therefore miss the programme guide for
those channels.

To fix this, the workflow adapts the EPG only (playlist files are never
modified beyond the EPG URL):

- EPG channel ids that match a playlist `tvg-id` exactly are kept.
- EPG channel ids whose normalized form (case-, whitespace- and
  hyphen-insensitive) uniquely matches exactly one playlist `tvg-id` are
  renamed, and all `<programme>` references are remapped.
- Playlist-style names (M3U display names, `tvg-name`, TXT channel names) are
  added as alias `<display-name>` entries so name-based players also match.
- Duplicate EPG channel definitions are merged.
- The gzip output is written with `mtime=0` so unchanged content produces
  byte-identical files and no spurious commits.

Only exact and normalized matching is used. No fuzzy matching. Channels that
the EPG source does not provide at all (mostly local channels) simply stay
without a programme guide.

---

## 12. Initial Commit

After creating the workflow and `.gitattributes`:

```bash
git add \
  .github/workflows/update.yml \
  .gitattributes

git commit -m "feat: initialize radio-clash"
git push -u origin main
```

Do not create or commit a README.

If the files already exist, inspect their current contents first and avoid unnecessary duplicate commits.

When moving the mirrored files from the repository root into `lyrics/`, use:

```bash
mkdir -p lyrics
git mv result.m3u result.txt ipv4.m3u ipv4.txt epg.gz lyrics/
```

so git records the moves as renames.

---

## 13. GitHub Actions Write Permission

The workflow needs permission to commit and push updates.

The workflow already declares:

```yaml
permissions:
  contents: write
```

If the first run fails during `git push`, inspect:

```text
Settings
→ Actions
→ General
→ Workflow permissions
```

Ensure that:

```text
Read and write permissions
```

is allowed.

If repository policy already honors the workflow-level `contents: write`, no additional change is necessary.

Do not create a personal PAT merely to bypass this unless explicitly requested by the user.

---

## 14. Trigger the First Update Manually

After pushing the workflow:

```bash
gh workflow run "Update Lyrics"
```

List recent runs:

```bash
gh run list \
  --workflow "Update Lyrics" \
  --limit 5
```

Inspect the latest run:

```bash
gh run view <RUN_ID>
```

If it fails:

```bash
gh run view <RUN_ID> --log-failed
```

Continue fixing issues until the first workflow run succeeds.

---

## 15. Verify the First Successful Sync

Confirm that the `lyrics/` directory contains:

```text
lyrics/result.m3u
lyrics/result.txt
lyrics/ipv4.m3u
lyrics/ipv4.txt
lyrics/epg.gz
```

Check file sizes:

```bash
ls -lh \
  lyrics/result.m3u \
  lyrics/result.txt \
  lyrics/ipv4.m3u \
  lyrics/ipv4.txt \
  lyrics/epg.gz
```

Inspect the M3U headers:

```bash
head -n 5 lyrics/result.m3u
head -n 5 lyrics/ipv4.m3u
```

They should begin with:

```text
#EXTM3U
```

and contain at least one:

```text
#EXTINF:
```

Validate EPG:

```bash
gzip -t lyrics/epg.gz
```

It must succeed.

---

## 16. Verify EPG URL Rewriting

Run:

```bash
grep -n 'x-tvg-url\|url-tvg\|epg.gz' lyrics/result.m3u | head -n 20
grep -n 'x-tvg-url\|url-tvg\|epg.gz' lyrics/ipv4.m3u | head -n 20
```

If an EPG URL is present, it should resolve to a URL similar to:

```text
https://ghfast.top/https://raw.githubusercontent.com/<USER>/radio-clash/main/lyrics/epg.gz
```

The following should no longer appear:

```text
https://guovin.github.io/iptv-api/epg.gz
```

Optionally verify the EPG channel adaptation:

```bash
zcat lyrics/epg.gz | grep -oE '<channel id="[^"]*"' | grep CCTV | head
```

CCTV channel ids should use the playlist style (for example `CCTV-1`), and
CCTV channels should carry both `CCTV1` and `CCTV-1` display-names.

---

## 17. Verify Raw URLs

Determine the GitHub username:

```bash
GH_USER="$(gh api user --jq .login)"
```

Define:

```bash
BASE="https://raw.githubusercontent.com/${GH_USER}/radio-clash/main/lyrics"
```

Test:

```bash
curl -fL "${BASE}/result.m3u" | head
curl -fL "${BASE}/result.txt" | head
curl -fL "${BASE}/ipv4.m3u" | head
curl -fL "${BASE}/ipv4.txt" | head
curl -fL "${BASE}/epg.gz" -o /tmp/radio-clash-epg.gz
gzip -t /tmp/radio-clash-epg.gz
```

All must succeed.

---

## 18. Verify Proxy URLs

Define:

```bash
GH_USER="$(gh api user --jq .login)"
PROXY="https://ghfast.top/"
RAW_BASE="https://raw.githubusercontent.com/${GH_USER}/radio-clash/main/lyrics"
```

Test:

```bash
curl -fL "${PROXY}${RAW_BASE}/result.m3u" | head
curl -fL "${PROXY}${RAW_BASE}/result.txt" | head
curl -fL "${PROXY}${RAW_BASE}/ipv4.m3u" | head
curl -fL "${PROXY}${RAW_BASE}/ipv4.txt" | head
```

Test EPG:

```bash
curl -fL \
  "${PROXY}${RAW_BASE}/epg.gz" \
  -o /tmp/radio-clash-proxy-epg.gz

gzip -t /tmp/radio-clash-proxy-epg.gz
```

If the proxy itself is inaccessible from the current development network but the repository and direct Raw URLs work correctly, do not alter the repository design.

Record the proxy test failure and allow the user to choose another compatible Raw proxy later.

---

## 19. Final URLs to Report to the User

At completion, print the actual usable URLs with the real GitHub username.

Do not leave `<USER>` placeholders.

Output:

```text
Full M3U
https://ghfast.top/https://raw.githubusercontent.com/ACTUAL_USER/radio-clash/main/lyrics/result.m3u

Full TXT
https://ghfast.top/https://raw.githubusercontent.com/ACTUAL_USER/radio-clash/main/lyrics/result.txt

IPv4 M3U
https://ghfast.top/https://raw.githubusercontent.com/ACTUAL_USER/radio-clash/main/lyrics/ipv4.m3u

IPv4 TXT
https://ghfast.top/https://raw.githubusercontent.com/ACTUAL_USER/radio-clash/main/lyrics/ipv4.txt

EPG
https://ghfast.top/https://raw.githubusercontent.com/ACTUAL_USER/radio-clash/main/lyrics/epg.gz
```

Also print the equivalent direct Raw URLs for troubleshooting:

```text
https://raw.githubusercontent.com/ACTUAL_USER/radio-clash/main/lyrics/result.m3u
https://raw.githubusercontent.com/ACTUAL_USER/radio-clash/main/lyrics/result.txt
https://raw.githubusercontent.com/ACTUAL_USER/radio-clash/main/lyrics/ipv4.m3u
https://raw.githubusercontent.com/ACTUAL_USER/radio-clash/main/lyrics/ipv4.txt
https://raw.githubusercontent.com/ACTUAL_USER/radio-clash/main/lyrics/epg.gz
```

---

## 20. Acceptance Criteria

The task is complete only when all of the following are true:

- [ ] The existing repository `Shawnkurt/radio-clash` is used as the target.
- [ ] The default branch is `main`.
- [ ] The repository is not a fork of the upstream project.
- [ ] No README file exists.
- [ ] `.github/workflows/update.yml` exists.
- [ ] `.gitattributes` exists.
- [ ] The workflow is named `Update Lyrics`.
- [ ] The workflow supports manual execution.
- [ ] The workflow runs automatically every 24 hours.
- [ ] `lyrics/result.m3u` is mirrored.
- [ ] `lyrics/result.txt` is mirrored.
- [ ] `lyrics/ipv4.m3u` is mirrored.
- [ ] `lyrics/ipv4.txt` is mirrored.
- [ ] `lyrics/epg.gz` is mirrored (from `https://epg.zsdc.eu.org/t.xml.gz`).
- [ ] All five files are handled as one atomic batch.
- [ ] Download failures preserve the previous known-good batch.
- [ ] Validation failures preserve the previous known-good batch.
- [ ] The upstream GitHub Pages EPG URL is rewritten in M3U files.
- [ ] Actual IPTV stream URLs are not rewritten.
- [ ] EPG channel ids and display-names are adapted to the playlist naming (exact + normalized matching only, EPG side only).
- [ ] EPG adaptation does not modify the playlist files.
- [ ] No commit is created when mirrored content is unchanged.
- [ ] The first manually triggered workflow run succeeds.
- [ ] Direct Raw URLs work.
- [ ] Final TV-facing proxy URLs are printed.
- [ ] Direct Raw URLs are printed for troubleshooting.

---

## 21. Do Not Add Extra Components

Unless explicitly requested by the user, do not:

- create another GitHub repository;
- rename or replace `Shawnkurt/radio-clash`;
- deploy a server;
- deploy Docker;
- deploy Cloudflare Workers;
- use OSS/COS;
- fork `Guovin/iptv-api`;
- modify the upstream project;
- regenerate IPTV sources locally;
- proxy IPTV video streams;
- create a database;
- add a web UI;
- add a README;
- add unnecessary dependencies;
- scaffold a Node.js or Python application;
- create a personal access token;
- automatically rotate between multiple third-party GitHub proxies.

Keep the repository intentionally minimal:

```text
GitHub Actions
+
.gitattributes
+
lyrics/ (five mirrored output files)
```

---

## 22. Maintenance Notes

If `ghfast.top` stops working, only change:

```yaml
GH_PROXY_PREFIX: "https://NEW_PROXY_PREFIX/"
```

and rerun the workflow.

Because the EPG URL embedded in both M3U files is generated from `GH_PROXY_PREFIX`, the next successful update will automatically embed the new EPG proxy URL.

If the EPG source changes, only change:

```yaml
EPG_SOURCE_URL: "https://NEW_EPG_SOURCE/t.xml.gz"
```

and rerun the workflow. The channel-name adaptation is data-driven and will
re-align itself against the playlists automatically.

The TV-facing playlist URLs will also need their proxy prefix changed, but the stable underlying Raw URLs remain:

```text
https://raw.githubusercontent.com/<USER>/radio-clash/main/lyrics/...
```

Therefore:

- `radio-clash` is the stable mirror layer;
- the third-party GitHub proxy is only a replaceable access layer;
- the EPG source is only a replaceable data layer.
