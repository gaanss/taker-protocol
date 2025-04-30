# Taker Protocol Light Mining Bot

This project automates registration, Twitter binding, and periodic mining tasks for Taker Light Mining.

## Setup

1. Create and activate a virtual environment:
   ```bash
   python -m venv venv
   source venv/bin/activate   # on Linux/macOS
   venv\Scripts\activate      # on Windows
   ```

2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

3. Prepare input files in `data/` folder (create this folder if missing):
   - `data/private_key.txt`: One Ethereum private key per line
   - `data/proxy.txt`: One proxy in `user:pass@ip:port` format per line
   - `data/twitter_token.txt`: One Twitter auth token per line
   - `data/invite.txt`: One invitation code per line

4. Ensure the `results/` directory exists (the script will create it if missing).

## Logging
- Uses `loguru` for colorized console logs.
- Default level is `INFO`, set `LOG_LEVEL=DEBUG` for full HTTP request/response tracing.

## Configuration Overrides
- Settings in `config.py` can be overridden via `settings.yaml`:
  ```yaml
  start_delay_sec: 10    # Delay before each registration/bind in seconds
  concurrency: 3         # Max concurrent mining jobs
  ```

## Usage
### Interactive CLI
Instead of running individual task scripts, use the main menu:
```bash
python main.py
```
This presents options for Registration, Twitter Binding, Mining Scheduler, and Exit.

### Registration Task
```bash
python main.py        # choose "1) Registration"
```
- Reads `private_key.txt` and `invite.txt` (one invitation code per line).
- Sleeps `start_delay_sec` seconds between accounts.
- Injects `invitationCode` from `invite.txt` into the login request.
- Populates `taker.db` and writes successes/failures to `results/registration_success.txt` or `results/registration_fail.txt`.

### Twitter Binding Task
```bash
python main.py        # choose "2) Twitter Binding"
```
- Reads `private_key.txt` and selects DB entries with valid JWT.
- Filters out tokens listed in `results/used_tokens.txt`, then iterates through remaining tokens, removing bad ones on the fly.
- Sleeps `start_delay_sec` seconds between accounts.
- Skips already-linked accounts by checking `twId` via API.
- On success writes to `results/twitter_success.txt` and appends the token to `results/used_tokens.txt`.

### Mining Scheduler
```bash
python main.py        # choose "3) Mining Scheduler"
```
- Runs asynchronous mining jobs on a random 25–27h interval per account, staggered by `start_delay_sec`.
- Concurrency controlled by `concurrency` in `settings.yaml`.
- Press `Ctrl+C` to gracefully stop the scheduler.

## Configurations

All key constants (file paths, endpoints, intervals) are in `config.py`.

## Coding Style

- Follows PEP 8 guidelines.
- Uses modular code in `utils/` and `tasks/`.

## License

MIT 