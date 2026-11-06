# Docker-Based HTML Cleanup

## Overview

This project documents a Docker-based workflow for processing a local copy of a website. A Python script was run inside a Docker container to scan HTML files and remove page elements identified by the IDs `header`, `sidebar`, and `footer`.

The workflow includes preparing the Docker environment, building the image, running the Python script, reviewing the cleanup results, accessing the output directory, and stopping Docker Desktop on macOS.

## Environment

- **Host shell:** zsh on macOS
- **Docker version:** 29.8.0, build 88096ef
- **Docker runtime:** Docker Desktop using the `docker:desktop-linux` builder
- **Docker base image:** `python:3.12-slim`
- **Python package:** `beautifulsoup4`
- **Project directory:** `html-editor`
- **Website input directory:** `websitename-copy`
- **Processing script:** `edit_html.py`
- **Docker image tag:** `html-editor`
- **Output directory:** `websitename-cleaned`

## Project Structure

```text
html-editor/
├── Dockerfile
├── edit_html.py
├── websitename-copy/
└── websitename-cleaned/
```

The output directory is created by the processing script, according to the terminal output.

## Commands and What They Do

### 1. Check Docker

```bash
docker --version
```

Checks whether Docker is installed and available in the terminal, and prints its version.

**Observed result:** Docker version 29.8.0, build 88096ef.

### 2. Create the project directory

```bash
mkdir html-editor
cd html-editor
```

- `mkdir html-editor` creates the project workspace.
- `cd html-editor` changes the current shell directory to the new project directory.

### 3. Create or edit the Dockerfile

```bash
nano Dockerfile
```

Opens the Nano text editor to create or edit the Dockerfile. The Dockerfile defines how the Docker image is built.

The supplied terminal transcript confirms that the Dockerfile was edited, but does not include its exact contents.

### 4. Build the Docker image

```bash
docker build -t html-editor .
```

Builds a Docker image from the Dockerfile in the current directory.

- `docker build` starts the image build process.
- `-t html-editor` assigns the image the name `html-editor`.
- `.` specifies the current directory as the build context.

**Observed result:** The build completed successfully. The output showed the `python:3.12-slim` base image and installation of `beautifulsoup4`.

### 5. Create or edit the Python script

```bash
nano edit_html.py
```

Opens the Python script in Nano. The script performs HTML-file discovery and cleanup.

The exact source code was not included in the terminal transcript, so its full implementation cannot be verified from the available information.

### 6. Inspect the project directory

```bash
ls
```

Lists files and folders in the current directory.

The terminal output showed:
- `Dockerfile`
- `edit_html.py`
- `websitename-copy`

### 7. Run the file-discovery pass in Docker

```bash
docker run --rm -v "$(pwd):/app" html-editor python edit_html.py
```

Runs `edit_html.py` in a temporary container created from the `html-editor` image.

Command explanation:

- `docker run` starts a container.
- `--rm` removes the container automatically after it exits.
- `-v "$(pwd):/app"` mounts the current host directory into the container at `/app`.
- `html-editor` specifies the Docker image.
- `python edit_html.py` executes the Python script inside the container.

**Observed result:** The first recorded run reported that it found 354 HTML files and printed their paths.

The listed paths included pages under `websitename-copy/deutsch.websitename.com/en`, including grammar, vocabulary, writing, and exercise pages.

### 8. Return to the project directory and revise the script

```bash
cd ~/html-editor
nano edit_html.py
```

- `cd ~/html-editor` returns the host shell to the project directory under the user's home directory.
- `nano edit_html.py` reopens the processing script for editing.

The transcript confirms that the script was edited again, but does not show the exact code changes.

### 9. Run the cleanup pass

```bash
docker run --rm -v "$(pwd):/app" html-editor python edit_html.py
```

Runs the updated script in a fresh temporary container using the same mounted project directory.

**Observed terminal output:**

- 354 HTML files found
- Cleanup started
- Cleanup completed
- 353 elements with `id='header'` removed
- 353 elements with `id='sidebar'` removed
- 353 elements with `id='footer'` removed
- Cleaned files saved in `/app/websitename-cleaned`

The log does not explain why each removal count was 353 rather than 354.

## Processing Workflow

1. **Prepare the workspace:** A dedicated `html-editor` directory was created for the project files and website copy.
2. **Build the runtime image:** Docker built an image called `html-editor` using the Dockerfile. The build output confirms Python 3.12 slim as the base image and installation of Beautiful Soup 4.
3. **Create and revise the script:** The `edit_html.py` file was opened in Nano more than once. The first execution discovered the HTML files; a later edit was followed by the cleanup execution.
4. **Discover the input files:** The script reported finding 354 HTML files.
5. **Clean the HTML pages:** The second execution reported that cleanup started and completed, with 353 removals for each of the three target IDs.
6. **Write the output:** The script reported saving the cleaned files in `/app/websitename-cleaned`.

## Final Result

The final successful processing run reported:

| Result | Reported value |
|---|---:|
| HTML files discovered | 354 |
| Header elements removed | 353 |
| Sidebar elements removed | 353 |
| Footer elements removed | 353 |
| Cleanup status | Cleanup complete! |
| Reported output path | `/app/websitename-cleaned` |

The counts confirm 353 removals for each of the three target IDs. They do not establish that all 354 files contained all three elements.

## Accessing the Output

The terminal transcript shows an attempt to run:

```bash
cd /app/websitename-cleaned
```

This failed because `/app` is the mount point inside the Docker container, not a directory path directly available in the host macOS shell.

The Docker command mounted the current host project directory at `/app` inside the container. Assuming the project is located at `~/html-editor`, use the following commands on the Mac:

```bash
cd ~/html-editor/websitename-cleaned
ls
```

- `cd` changes to the host-side output directory.
- `ls` lists the cleaned files.

The transcript does not show a successful listing of the output directory, so the output contents were not independently verified in the recorded session.

## Stopping Docker Desktop on macOS

After the cleanup and verification are complete, Docker Desktop can be stopped.

### Option 1: Using Terminal (recommended)

```bash
osascript -e 'quit app "Docker"'
```

This command asks macOS to quit the Docker Desktop application.

### Option 2: Using Docker Desktop UI

1. Click the Docker whale icon in the macOS menu bar.
2. Select **Quit Docker Desktop**.

### Verify Docker has stopped

Run this command in the macOS host terminal:

```bash
docker info >/dev/null 2>&1 && echo "Docker is running" || echo "Docker is not running"
```

Expected output after Docker Desktop has fully stopped:

```text
Docker is not running
```

The command suppresses the normal output and error messages from `docker info`. It checks whether the Docker CLI can communicate with the Docker daemon. If Docker Desktop is still shutting down, wait briefly and run the check again.

### Important Docker Notes

- Quitting Docker Desktop stops the Docker daemon and its running containers.
- The `--rm` option removes the temporary container after it exits. It does **not** remove the `html-editor` Docker image.
- The cleaned HTML files are stored in the mounted host project directory. Quitting Docker Desktop does not delete these files.
- Start Docker Desktop again and wait for its engine to become available before running further Docker commands.

## Notes and Limitations

- This README is based on the supplied terminal transcript and technical report information.
- The exact contents of `Dockerfile` and `edit_html.py` were not included.
- The log confirms that the Docker image built successfully and that the script printed `Cleanup complete!`.
- The log does not include a before-and-after HTML comparison.
- No browser-based validation or confirmation that all pages render correctly after cleanup was included.
- The log reports 354 discovered HTML files and 353 removals for each target ID. It does not explain the difference.
- The output directory was reported by the script, but its contents were not independently verified in the recorded session.

## Conclusion

A Docker-based Python environment was successfully prepared and used to process a local collection of website HTML pages. The processing script found 354 HTML files and completed a cleanup operation that reported removing the header, sidebar, and footer elements 353 times each.

The cleaned output was directed to `websitename-cleaned`. The only visible command-line issue occurred afterward when a container-internal path was used from the host macOS shell. This path issue does not indicate that the cleanup operation itself failed.

The workflow can be completed by checking the output directory on the host Mac and quitting Docker Desktop when no further container work is required.

---

*End of README*
