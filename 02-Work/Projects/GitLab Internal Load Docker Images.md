# GitLab Internal - Load Docker Images

## Infrastructure Overview

| Role | IP Address | Notes |
|------|-----------|-------|
| GitLab Web Server | `10.51.24.10` | |
| Runner (tagged jobs) | `10.51.26.5` | Only picks up tagged jobs; has uplift connection to NGA |
| Runner | `10.51.26.7` | No outside access; runs normal jobs |
| Runner | `10.51.26.11` | No outside access; runs normal jobs |
| Unused | `10.51.26.26` | Future plan: Docker registry and PyPI mirror |

## Network Constraints

GitLab Internal **does not have outside internet access** but does have pull access from GitLab Enterprise. All referenced Docker images must be hosted in GitLab Enterprise and then loaded into the runners.

> [!note] Future Plans
> We may be able to proxy traffic through GitLab Enterprise, or use `10.51.26.26` as a Docker registry and PyPI mirror.

## Image Loading Process

Base images and scanning products are tarred together and loaded into the package registry of the [`pull-scanner-images`](https://gitlab.761link.net/enterprise/pull-scanner-images) project.

### Scheduled Branches

Two branches run on a schedule to build and publish the image tarballs:

| Branch | Schedule | Link |
|--------|----------|------|
| `weekly-updates` | Every Tuesday at 2:00 AM | [View Branch](https://gitlab.761link.net/enterprise/pull-scanner-images/-/tree/weekly-updates?ref_type=heads) |
| `monthly-updates` | 1st of every month at 2:00 AM | [View Branch](https://gitlab.761link.net/enterprise/pull-scanner-images/-/tree/monthly-updates?ref_type=heads) |

### Runner Cron Jobs

On each runner (`10.51.26.5`, `10.51.26.7`, `10.51.26.11`), two cron jobs pull the images 3 hours after the CI pipelines complete:

![[Pasted image 20260213091403.png]]

The crontab entries are:

```cron
# m  h  dom  mon  dow  command
0  5  *  *  2  bash /home/robert.smith/pull_weekly_images.sh >> /home/robert.smith/image_log.txt
0  5  1  *  *  bash /home/robert.smith/pull_monthly_images.sh >> /home/robert.smith/image_log.txt
```

| Job | Schedule | Script |
|-----|----------|--------|
| Weekly | Every Tuesday at 5:00 AM | `/home/robert.smith/pull_weekly_images.sh` |
| Monthly | 1st of every month at 5:00 AM | `/home/robert.smith/pull_monthly_images.sh` |

Both jobs append output to `/home/robert.smith/image_log.txt`.

## Authentication

The bash scripts authenticate using a **Personal Access Token (PAT)** managed at:
[pull-scanner-images Access Tokens](https://gitlab.761link.net/enterprise/pull-scanner-images/-/settings/access_tokens)

> [!warning] Token Rotation
> **Next rotation date:** June 25, 2026 at 7:00:00 PM CDT

## Script Breakdown (`pull_weekly_images.sh`)

![[Pasted image 20260213091505.png]]

The script performs the following steps:

```bash
# 1. Download the tarball from the GitLab Package Registry
wget --header="PRIVATE-TOKEN: <PAT>" \
  -O "https://gitlab.761link.net/api/v4/projects/2435/packages/generic/images/latest/weekly_updates.tar.gz"

# 2. Define scanner images that also need to be pulled from the enterprise registry
docker_images=(
    "gitlab-registry.761link.net/scanners/clam-av:latest"
    "gitlab-registry.761link.net/scanners/syft:latest"
)

# 3. Extract the tarball
tar -zxvf weekly_updates.tar.gz
cd image

# 4. Load each .tar image file into Docker
for image in $(ls); do docker load -i ${image}; done
cd ..

# 5. Cleanup - remove extracted folder and tarball
rm -rf image
rm -rf weekly_updates.tar.gz

# 6. Pull additional scanner images from the enterprise registry
for image in "${docker_images[@]}"; do
    docker pull "$image"
done
```

### Script Workflow Summary

1. **Download** - Uses `wget` with a PAT header to download `weekly_updates.tar.gz` from the GitLab Package Registry API (project ID `2435`)
2. **Extract** - Unpacks the tarball into an `image/` directory containing individual Docker image `.tar` files
3. **Load** - Iterates over every file in the `image/` directory and runs `docker load -i` to import each image into the local Docker daemon
4. **Cleanup** - Removes the extracted `image/` folder and the `.tar.gz` archive
5. **Pull Registry Images** - Pulls the latest `clam-av` and `syft` scanner images directly from `gitlab-registry.761link.net`
