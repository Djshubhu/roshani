# Automated Web Scanner

A Python Jupyter Notebook that checks selected website security headers and Cross-Origin Resource Sharing (CORS) settings. It displays observed values, identifies configurations that need review, and suggests improvements.

## Student details

| Field | Details |
|---|---|
| Student name | Roshni Pagariya |
| Class | TYBSc Forensic Science |
| Institution | Sandip University |
| Submission date | 27 October 2026 |
| Programme | IBM SkillsBuild Cyber Security Bootcamp 2026, conducted by BharatCares |

## Project files

| File | Purpose |
|---|---|
| `RoshniPagariya_AutomatedWebScanner.ipynb` | Complete project code, explanations, local demonstration, saved outputs, and tests |
| `requirements.txt` | Packages for the local Jupyter interface and Python kernel |
| `README.md` | Project overview and setup instructions |

The notebook is named for final submission. The code does not depend on the notebook filename. The project report and full student identification details are supplied separately for academic submission.

## What the scanner checks

- Whether the supplied URL uses HTTPS.
- HSTS on HTTPS responses, including a basic positive `max-age` check.
- Content-Security-Policy presence and selected permissive keywords.
- Framing protection through CSP `frame-ancestors` or X-Frame-Options.
- X-Content-Type-Options and Referrer-Policy values.
- Permissions-Policy presence.
- CORS responses to two unrelated test origins and the `null` origin.
- Reflected origins, wildcard CORS, credential permission, and selected cache-variation concerns.

For a successful target, the scanner sends one baseline GET request and three additional GET requests with different `Origin` headers. It collects response headers without downloading the response body. It does not send authentication credentials, crawl links, or follow redirects.

## Requirements

- Python **3.9 or newer**.
- Jupyter Notebook, JupyterLab, or a compatible notebook environment such as Google Colab.
- Permission to start a temporary loopback server for the built-in demonstration.
- Internet access when checking an online target.

The scanner uses only Python standard-library modules. The supplied requirements file installs `notebook>=7,<8` and `ipykernel>=6,<7` for local notebook execution. No API key, password, or external dataset is needed.

## Local setup

1. Keep the project files together in one folder.
2. Open a terminal in that folder.
3. Install the notebook tools:

   ```bash
   python -m pip install -r requirements.txt
   ```

4. Start Jupyter:

   ```bash
   jupyter notebook
   ```

5. Open the `.ipynb` file and select **Run All**.

On systems where Python is named `python3`, use `python3` instead of `python`. If a compatible Jupyter environment is already installed, a separate installation may be unnecessary.

## Run the demonstration

No user input is required. Running all cells starts a temporary HTTP server on `127.0.0.1` and scans three local pages:

| Page | Configuration |
|---|---|
| `/weak` | Missing selected security headers; reflects supplied origins with credential permission |
| `/hardened` | Restrictive example headers; no CORS permissions |
| `/public` | Public wildcard CORS without credentials; other security headers omitted |

The demonstration prints the results, stops the server, and runs ten automated tests. Its port is selected automatically and can change on each run. All pages use HTTP, so the HTTPS review is expected even for `/hardened`.

Public website scanning is disabled by default.

## Input for an authorized website scan

After running the earlier cells, edit the configuration cell near the end:

```python
RUN_AUTHORIZED_SCAN = True
TARGET_URLS = ["https://your-authorized-host.example/path"]
TIMEOUT_SECONDS = 8
REQUEST_DELAY_SECONDS = 0.25
EXPORT_FILES = True
```

Replace the placeholder address with a website or endpoint you own or have permission to audit. Then run the final scan cell. The example address above is not a working target.

| Setting | Accepted input |
|---|---|
| `RUN_AUTHORIZED_SCAN` | `True` to run the optional scan; default `False` |
| `TARGET_URLS` | A list containing 1 to 10 distinct complete HTTP or HTTPS URLs |
| `TIMEOUT_SECONDS` | Greater than 0 and no more than 60 seconds |
| `REQUEST_DELAY_SECONDS` | Between 0 and 10 seconds before each CORS probe |
| `EXPORT_FILES` | `True` to save JSON and CSV; default `False` |

URLs must include `http://` or `https://`. URLs with embedded credentials, fragments (`#...`), invalid ports, or whitespace are rejected. Duplicate URLs are removed. Targets run sequentially. Redirect destinations must be entered separately if they are also authorized for review.

**Important:** Running **Run All** again restores the settings written in the configuration cell. Save your desired values in that cell before rerunning the notebook.

## Outputs

The notebook displays each target URL, HTTP status, request count, finding counts, observed evidence, and recommendations for review items and errors.

| Label | Meaning |
|---|---|
| `PASS` | A specific check passed |
| `PRESENT` | A header or directive exists but needs deeper review |
| `REVIEW` | A configuration concern needs examination |
| `INFO` | Contextual information |
| `ERROR` | A request failed |

These labels are not vulnerability severity levels or security grades. A missing header is not automatically exploitable. Public wildcard CORS can be intentional, and wildcard CORS with credentials is blocked for credentialed browser reads. Reflected origins require manual review of the allowlist, data sensitivity, and browser behavior.

### Optional exports

With `EXPORT_FILES = True`, the authorized scan writes timestamped files to `scanner_results/` under the notebook kernel's working directory:

- **JSON:** Complete results, including baseline and probe headers, status, timing, errors, and findings.
- **CSV:** One row per finding with the target URL, UTC timestamp, check, status, evidence, and recommendation.

The folder is normally beside the notebook when Jupyter is started from the project folder. Treat exported metadata as potentially sensitive. No persistent exports are created by default.

## Data source

**External dataset link: Not applicable.** The scanner uses HTTP responses from the exact supplied endpoints. Its reproducible demonstration data is generated by `DemoHandler` inside the notebook; no separate dataset download is required.

The probe origins `https://audit-origin-a.invalid`, `https://audit-origin-b.invalid`, and `null` are request-header values. They are not destinations that the scanner visits.

## Verification and saved results

The supplied notebook was executed in a Python 3.9 Jupyter kernel on macOS. Its format was validated, all code cells completed, and all **10 automated tests passed**.

Tests cover the local requests, origin reflection, hardened headers, public wildcard CORS, redirects and HTTP errors, invalid URLs, wildcard CORS with credentials, HSTS analysis, simulated request failure, and JSON/CSV export round trips.

The saved demonstration produced these counts:

| Page | REVIEW | INFO | PRESENT | PASS | ERROR |
|---|---:|---:|---:|---:|---:|
| `/weak` | 11 | 2 | 0 | 0 | 0 |
| `/hardened` | 1 | 4 | 3 | 2 | 0 |
| `/public` | 5 | 5 | 0 | 0 | 0 |

Counts include overlapping observations and do not represent distinct exploitable vulnerabilities. No public website was audited in the saved demonstration. HSTS analysis tests use supplied observations; the local server does not demonstrate live TLS behavior.

## Limitations

- Results cover only the exact endpoints and request conditions tested.
- There is no cookie audit, OPTIONS preflight, authenticated scan, browser exploit validation, or automatic repair.
- CSP and other complex policies receive limited analysis; presence alone does not establish effective protection.
- HTTPS requests retain normal certificate verification, but TLS protocols, cipher suites, and certificate-expiry reporting are outside the audit.
- Redirects are not followed. Non-successful baselines do not trigger CORS probes.
- There is no web dashboard, numeric security score, HTML report, or full forensic chain-of-custody implementation.
- Some hosted environments restrict loopback servers or outbound connections; use a local Jupyter session if the demonstration is blocked.

A clean result does not certify that a website is secure. Use the findings as a starting point for manual investigation.

## Authorized use

Scan only systems you own or have explicit permission to audit. Keep requests within the approved scope and protect exported results. The tool does not bypass authentication, rate limits, or access controls.

## References

- [MDN CORS configuration](https://developer.mozilla.org/en-US/docs/Web/Security/Practical_implementation_guides/CORS)
- [MDN Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy)
- [MDN Strict-Transport-Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Strict-Transport-Security)
- [OWASP Secure Headers Project](https://owasp.org/projects/secure-headers-project)
- [Python urllib.request documentation](https://docs.python.org/3/library/urllib.request.html)
- [Python unittest documentation](https://docs.python.org/3/library/unittest.html)
