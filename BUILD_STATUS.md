# Nighty 2.3 build status

The repository is **not currently buildable or verified to run**. The Windows workflow is configured to fail at syntax validation until the source is repaired. A successful dependency installation alone does not establish application functionality.

## Verified on Windows with Python 3.11

- The public packages in `requirements.txt` and PyInstaller installed in an isolated Python 3.11 environment.
- Selected imports (`discord_protos`, `youtubesearchpython.SearchVideos`, `win10toast_click.ToastNotifier`, `tls_client.Session`) succeeded after pinning `setuptools<81` for `win10toast_click`.
- `python -m py_compile Nighty.py` still fails: an unclosed triple-quoted string is reported at line 10350. The actual damage begins earlier in the custom tab and HTML code around lines 2472–3557. In that section, Python indentation and f-strings are corrupted; later code is swallowed into a string. This cannot be safely fixed by adding a closing quote at the end.
- An actual PyInstaller one-file build was attempted on Windows. It stopped while analyzing `Nighty.py` with the same syntax error; no `Nighty.exe` was produced.
- Static parsing also fails for multiple files in the bundled `discord/` and `webview/` packages, including `discord/client.py`, `discord/utils.py`, `webview/__init__.py`, and `webview/platforms/winforms.py`. The workflow now checks those packages before building.
- `nightycore` and `discord_client` are imported but absent from this repository. No matching PyPI distribution was found for those exact names. Their source and license terms are needed before a working build can be verified. Do not substitute unrelated packages.

## Static security findings

- `Nighty.py` stores a Discord user token in plaintext under the user's application data directory (`config.json`, e.g. lines 2078–2081 and 2191–2216). It also includes code that reads a token from the Discord web client or desktop client (lines 2057–2076 and 10376–10401). The direct token and license-key log statements were removed, but plaintext storage and token extraction remain.
- The app sends its license key and machine UUID to `http://15.204.5.83` (lines 10249–10259), and fetches encrypted UI content from that host over HTTP (lines 3634–3651). It also loads pages from `nighty.one`. Remote content and the unavailable `nightycore` cannot be audited from this repository.
- Community scripts can be downloaded and later executed with `exec` (lines 1495–1518 and 1838–1877), and ZIP archives are extracted without member-path validation. These are code execution and path traversal risks if a source is untrusted.
- The `client_login` path terminates the Discord process, extracts a token via `discord_client`, and restarts Discord (lines 10376–10401). That module is missing, so its behavior cannot be audited.

No real Discord login, token use, or external application content was executed during this review. A release should wait for recovery of authentic source for the damaged files and the missing private modules, followed by syntax, build, offline UI, and end-to-end tests in a controlled environment.
