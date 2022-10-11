This script is a simple drop-in replacement for `rsync` for parallelising your data transfer.

Rsync is the tool of choice for copying/syncing data between locations.
It is capable of only transfering files which have changed and resuming upload/downloads.
However, the transfer speed of a single `rsync` can be somewhat slow.
This is a problem when transfering a large amount of data as it will take some time to complete.

If your rsync contains lots of files, you can benefit from transfering files in parallel.
Thus benfiting from a more effective use of your available network bandwidth and gettging the job done faster.

# Usage

If your `rsync` command looks like this:

```bash
rsync \
  --times --recursive --progress \
  --exclude "raw_reads" --exclude ".snakemake" \
  user@example.com:/my_remote_dir/ /my_local_dir/
```

Simply replace the `rsync` executable for this script:

```bash
./prsync \
  --times --recursive --progress \
  --exclude "raw_reads" --exclude ".snakemake" \
  user@example.com:/my_remote_dir/ /my_local_dir/
```

## Number of Parallel Jobs

By default, the script will use 1 parallel job for each processor on the machine.
This is determined by `nproc` and if this fails, we fall back to `10` parallel jobs for transfering files.
This behaviour can be overriden by using `--parallel` as the first command line argument to the script:

```bash
./prsync \
  --parallel=20 \
  --times --recursive --progress \
  --exclude "raw_reads" --exclude ".snakemake" \
  user@example.com:/my_remote_dir/ /my_local_dir/
```

# Implementation

The list of files to be transfered is calulated by first running `rsync` in dry-run mode.
It is then split into `N` chunks based on the value of `--parallel` (10 by default).
Each "chunk" of files is then passed to parallel `rsync` process.

To ensure a more balanced distribution of files among chunks, files are sorted by decreasing filesize and then assigned to the chunk with the least data to process.
This ensures that chunks are of approximately the same size and have the same number of files to process.
Thus parallel `rsync` processes will complete at around the same time.
