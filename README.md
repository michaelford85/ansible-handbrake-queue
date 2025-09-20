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
- Allow **per-job subtitle options** using simple shorthand fields.

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
  binary: "/opt/homebrew/bin/HandBrakeCLI"

  # Preset file (exported from GUI or UserPresets.json)
  preset_file: "./presets/bluray_and_dvd.json"

  # Exact preset name (check with: HandBrakeCLI --preset-list)
  preset_name: "Blu Ray and DVD"

  # Where logs should be stored (relative to the playbook directory)
  log_dir: "{{ playbook_dir }}/handbrake-logs"

  # Optional: pass additional CLI flags (applies to ALL jobs)
  # extra_args:
  #   - "--markers"

  jobs:
    # Standard encode
    - src: "/media/raw/MovieA.mkv"
      dest: "/media/transcoded/MovieA.mkv"

    # Chinese subs on track 2, set as default
    - src: "/media/raw/MovieB.mkv"
      dest: "/media/transcoded/MovieB.mkv"
      subtitle_track: 2
      subtitle_default: true

    # Russian subs on track 4, burned in and default
    - src: "/media/raw/MovieC.mkv"
      dest: "/media/transcoded/MovieC.mkv"
      subtitle_track: 4
      subtitle_burn: true
      subtitle_default: true

    # By language (English + Spanish)
    - src: "/media/raw/MovieD.mkv"
      dest: "/media/transcoded/MovieD.mkv"
      subtitle_lang_list: ["eng","spa"]
```

---

## ▶️ Running the Playbook

Run the playbook with colorized output:

```bash
ANSIBLE_FORCE_COLOR=1 ansible-playbook handbrake_queue.yml
```

---

## 📺 Monitoring Progress

HandBrake output is redirected to log files in `{{ playbook_dir }}/handbrake-logs/`.

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
- **Subtitle shorthand**: Use `subtitle_track`, `subtitle_burn`, `subtitle_default`, and `subtitle_lang_list` for per-job subtitle handling.
- **Parallel option**: If you want multiple encodes at once, remove `serial: 1` and add `async`/`poll` to the transcode task.
