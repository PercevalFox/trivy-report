# trivy-report

Turn verbose [Trivy](https://github.com/aquasecurity/trivy) JSON output into a
short, PR-friendly Markdown summary — a severity count table plus the
vulnerabilities that actually have a fix available, sorted worst-first.

Trivy's terminal output is great to read locally but noisy in a pull request or a
CI job summary. `trivy-report` gives you the 20% that matters: what's broken, how
bad it is, and which version fixes it.

## Why

- **Zero dependencies** — pure Python standard library, one file.
- **Prioritizes fixable issues** — surfaces vulnerabilities with an upstream fix,
  because those are the ones you can act on today.
- **CI gating** — `--fail-on HIGH` exits non-zero so a build can block on severe
  findings.
- **Pipes cleanly** — reads from a file or stdin.

## Install

No install needed — it's a single file. Clone and run:

```bash
git clone https://github.com/YOUR-USERNAME/trivy-report.git
cd trivy-report
```

Requires Python 3.7+.

## Usage

```bash
# From a file
trivy image -f json myapp:1.4.0 > scan.json
python trivy_report.py scan.json

# Straight from a pipe
trivy image -f json myapp:1.4.0 | python trivy_report.py

# Write to a file and gate CI on HIGH+ findings
python trivy_report.py scan.json -o report.md --fail-on HIGH
```

### Example output

```markdown
## 🛡️ Trivy scan report — `myapp:1.4.0`
_Type: container_image_

| Severity | Count |
| --- | --- |
| 🔴 CRITICAL | 1 |
| 🟠 HIGH | 2 |
| 🔵 LOW | 1 |
| **Total** | **4** |

### 🔧 Fixable vulnerabilities (3)

| Severity | Package | Installed | Fixed in | CVE |
| --- | --- | --- | --- | --- |
| 🔴 CRITICAL | `libc6` | 2.36-9 | **2.36-9+deb12u3** | CVE-2023-4911 |
| ...
```

Try it right now against the bundled sample:

```bash
python trivy_report.py examples/sample-trivy.json
```

## Use in GitHub Actions

A ready-to-use workflow is in [`.github/workflows/security-scan.yml`](.github/workflows/security-scan.yml).
It scans your image, renders the report, and posts it into the job summary:

```yaml
- name: Scan image
  run: trivy image -f json myapp:latest > scan.json

- name: Render report
  run: python trivy_report.py scan.json --fail-on CRITICAL >> "$GITHUB_STEP_SUMMARY"
```

## Options

| Flag | Description |
| --- | --- |
| `input` | Trivy JSON file (default: read from stdin) |
| `-o`, `--output` | Write the report to a file instead of stdout |
| `--fail-on SEVERITY` | Exit 1 if a vuln at or above this severity is found |

## Contributing

Questions and ideas are welcome in the **Discussions** tab. If `trivy-report`
helped you, a ⭐ is appreciated.

## License

MIT — see [LICENSE](LICENSE).
