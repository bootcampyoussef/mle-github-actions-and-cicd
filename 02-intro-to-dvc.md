# Intro to Data Version Control (DVC)

[DVC](https://dvc.org/) brings version control workflows to data and model artifacts. While Git tracks code and small metadata files, DVC stores large files in a cache or remote storage location.

In this lesson, you will track one Parquet dataset with DVC and push the actual data file to a local DVC remote directory.

## Why DVC helps

Machine learning projects change in two dimensions: code changes and data changes. Git handles code well, but large data files do not belong in Git history. DVC keeps a small `.dvc` metadata file in Git so a teammate or CI workflow can recover the exact data version later.

You can use DVC to:

- track large data files without committing them to Git,
- restore the same dataset version on another machine,
- compare metrics across experiments,
- define reproducible data processing pipelines.

## Data versioning workflow

```mermaid
flowchart LR
    A["Download data"] --> B["dvc add data file"]
    B --> C["git add .dvc<br>metadata"]
    C --> D["dvc push<br>to local remote"]
    D --> E["Teammate or CI<br>runs dvc pull"]
```

## Track the taxi dataset

Download the Green Taxi data for January 2025:

`macOS` / `Linux` / `Git Bash`:

```bash
mkdir -p data
curl -L --fail -o data/green_tripdata_2025-01.parquet \
  https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2025-01.parquet
```

`PowerShell`:

```powershell
New-Item -ItemType Directory -Force data
curl.exe -L --fail -o data\green_tripdata_2025-01.parquet `
  https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2025-01.parquet
```

Initialize DVC in the repository:

```bash
dvc init
```

Add the local data file to DVC:

```bash
dvc add ./data/green_tripdata_2025-01.parquet
```

DVC creates a small metadata file named `data/green_tripdata_2025-01.parquet.dvc`. Add that file to Git:

```bash
git add .dvc/.gitignore .dvc/config .dvcignore data/green_tripdata_2025-01.parquet.dvc
git commit -m "Track taxi data with DVC"
```

The Parquet file itself stays out of Git. The `.dvc` file records enough information for DVC to find and verify the file through its cache or remote.

## Add a local remote

Create a directory outside the repository to act as the DVC remote:

`macOS` / `Linux` / `Git Bash`:

```bash
mkdir -p ../dvc-local-remote
```

`PowerShell`:

```powershell
New-Item -ItemType Directory -Force ../dvc-local-remote
```

Add that directory as the default DVC remote:

```bash
dvc remote add -d localremote ../dvc-local-remote
```

Commit the shared remote configuration:

```bash
git add .dvc/config
git commit -m "Add local DVC remote"
```

Push the data file to the local remote:

```bash
dvc push
```

Now another team member or a GitHub Actions workflow can clone the repository and recover the data with:

```bash
dvc pull
```

Because this is a local-first setup, the remote is just another directory on your machine. For team projects, you can later replace it with shared storage after the DVC workflow is clear.

When you clone the repository into another folder on the same machine, `dvc pull` can recover the Parquet file as long as the configured local remote path still points to the directory where you ran `dvc push`.

| File or directory | Commit to Git? | Why |
| --- | --- | --- |
| `data/green_tripdata_2025-01.parquet` | No | It is the large raw data file. |
| `data/green_tripdata_2025-01.parquet.dvc` | Yes | It is the small metadata pointer DVC uses to recover the data. |
| `.dvc/config` | Yes | It stores the shared local remote path for the exercise. |
| `../dvc-local-remote/` | No | It is the local storage area for DVC objects. |

## Inspect the metadata

Open `data/green_tripdata_2025-01.parquet.dvc` after running `dvc add`. Notice that Git tracks metadata such as the file path, hash and size, not the full Parquet contents. This is the key handoff between Git and DVC.

## Pipelines

DVC can also define data pipelines for filtering, feature engineering, and model training. A pipeline records commands, dependencies, outputs and metrics so the work can be reproduced. This repository only uses DVC data versioning, but the next natural step is the [DVC data pipelines guide](https://dvc.org/doc/start/data-pipelines).

## Exercise

- Inspect the generated `.dvc` file and explain which fields Git tracks.
- Run `dvc status` before and after changing the local Parquet file.
- After `dvc push`, inspect `../dvc-local-remote` and confirm DVC stored the data object there.
- Clone your fork into a separate folder, configure the same local remote path and test `dvc pull`.
