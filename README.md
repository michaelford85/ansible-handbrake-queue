# HandBrakeCLI Queue Playbook

## 📌 Purpose

This Ansible playbook automates transcoding video files with **HandBrakeCLI**.  
Instead of manually queueing jobs in the HandBrake GUI, you can define a list of source and destination files in a variable file.  
The playbook will:

- Loop through each job one at a time (queue-like behavior).
- Create destination directories if they don’t exist.
- Run `HandBrakeCLI` with your chosen preset.
- Skip jobs if the destination file already exists (idempotency).
- Capture the **live HandBrake output** into a log file so you can monitor progress with `tail -f`.

This is useful for **batch video transcoding** in a homelab, media server, or Jellyfin/Plex workflow.

---

## 📂 File Structure

```
handbrake_queue.yml   # The main Ansible playbook
jobs.yml              # Variable file with source/destination mappings
handbrake-logs/       # Log directory (auto-created)
```

---

## ⚙️ jobs.yml Example

```yaml
handbrake:
  # Path to HandBrakeCLI binary
  binary: "/Applications/HandBrake.app/Contents/MacOS/HandBrakeCLI"

  # Preset file (exported from GUI or UserPresets.json)
  preset_file: "/Volumes/Crucial X9/Jellyfin Working Directory/Handbrake Presets"

  # Exact preset name (check with: HandBrakeCLI --preset-list)
  preset_name: "Blu Ray and DVD"

  # Where logs should be stored
  log_dir: "./handbrake-logs"

  # Optional: pass additional CLI flags
  # extra_args:
  #   - "--encoder-preset"
  #   - "slow"
  #   - "--markers"

  jobs:
    - src: "/Volumes/Crucial X9/Jellyfin Working Directory/raw media/movies/The Accountant/The Accountant (2016).mkv"
      dest: "/Volumes/Crucial X9/Jellyfin Working Directory/transcoded media/The Accountant (2016) [tmdbid-302946]/The Accountant (2016) [tmdbid-302946].mkv"

    # Add more jobs as needed...
```

---

## ▶️ Running the Playbook

Run the playbook with colorized output:

```bash
ANSIBLE_FORCE_COLOR=1 ansible-playbook handbrake_queue.yml
```

---

## 📺 Monitoring Progress

HandBrake output is redirected to log files in `./handbrake-logs/`.

Open another terminal and run:

```bash
tail -f handbrake-logs/*.log
```

This will show live encoding progress, e.g.:

```
x265 [info]: tools: ...
Encoding: task 1 of 1, 10.18 % (16.47 fps, avg 15.51 fps, ETA 02h57m35s)
...
```

---

## 📝 Notes

- **Spaces in file paths**: No need to escape, `argv` handles them correctly.
- **Idempotent**: The playbook won’t re-encode a file if the destination already exists.
- **Serial processing**: Uses `serial: 1` to process one job at a time (like the GUI queue).
- **Parallel option**: If you want multiple encodes at once, remove `serial: 1` and add `async`/`poll` to the transcode task.
