# rsbu

{**rs**}ync {**b**}ack{**u**}p tool

## Description

`rsbu` is a [`rsync`][rsync-website] wrapper. It is designed to back up one directory to another without memorizing `rsync` options and quirks. In addition, `rsbu` configures `rsync` to use per-directory filter rules: `.rsync-exclude` for simple exclude patterns and `.rsync-filter` for more complex rules.

## Usage

```shell
rsbu [--print-rsync-command | --pretty-print-rsync-command] [EXTRA RSYNC OPTIONS ...] [--] SRC DEST
```

## How It Works

`rsbu` is essentially a shell script that builds and executes `rsync` command line. The command line is built as follows:

```
rsync_name_or_path non_filter_options options_from_args filter_options src_dir_from_args dest_dir_from_args
```

Use `--print-rsync-command` or `--pretty-print-rsync-command` to print the resulting command line instead of executing it.

### _rsync_name_or_path_

A name or a path of the `rsync` executable. The defaut value is `rsync`, can be overridden with the `RSBU_RSYNC` environment variable.

### _non_filter_options_

Any `rsync` options except for filter–related (`--filter`, `--include*`, `--exclude*`). The default value is `--delete --archive --hard-links --human-readable --progress`, can be overridden with the `RSYNC_OPTIONS` environment variable.

### _options_from_args_

Any additional options from `rsbu` command line arguments.

### _filter_options_

Filter–related `rsync` options (`--filter`, `--include*`, `--exclude*`). The default value is `--filter='+ .rsync-exclude' --filter='+ .rsync-filter' --filter='dir-merge,- .rsync-exclude' --filter='dir-merge .rsync-filter'`, can be overridden with the `RSYNC_FILTERS` environment variable.

### _src_dir_from_args_

A path of the source directory from `rsbu` command line arguments. If the path has no traling slash, it will be appended.

### _dest_dir_from_args_

A path of the backup directory from `rsbu` command line arguments.

## Examples

* Default options/filters, no extra options

  ```shell
  rsbu ~/Documents /media/user/ext/Documents --pretty-print-rsync-command
  ```

  ```shell
  rsync \
      --delete \
      --archive \
      --hard-links \
      --human-readable \
      --progress \
      --filter=+\ .rsync-exclude \
      --filter=+\ .rsync-filter \
      --filter=dir-merge\,-\ .rsync-exclude \
      --filter=dir-merge\ .rsync-filter \
      -- \
      /home/user/Documents/ \
      /media/user/ext/Documents
  ```

* Overridden options, default filters, extra options

  ```shell
  RSBU_OPTIONS='--foo --bar=2' rsbu --baz ~ /media/user/ext/home --pretty-print-rsync-command --qux
  ```

  ```shell
  rsync \
      --foo \
      --bar=2 \
      --baz \
      --qux \
      --filter=+\ .rsync-exclude \
      --filter=+\ .rsync-filter \
      --filter=dir-merge\,-\ .rsync-exclude \
      --filter=dir-merge\ .rsync-filter \
      -- \
      /home/user/ \
      /media/user/ext/home
  ```

* Default options, disabled filters, the custom `rsync` executable

  ```shell
  RSBU_RSYNC=/path/to/rsync-custom RSBU_FILTERS= rsbu ~/.config/ /media/user/ext/.config/ --pretty-print-rsync-command
  ```

  ```shell
  /path/to/rsync-custom \
      --delete \
      --archive \
      --hard-links \
      --human-readable \
      --progress \
      -- \
      /home/user/.config/ \
      /media/user/ext/.config/
  ```

# Filter Rules

For detailed information about filter rules, see [rsync(1)][rsync-man] → [FILTER RULES][rsync-man-filter-rules].

By default, `rsbu` instructs `rsync` to recursevely scan the source directory for two files containing filter rules: `.rsync-exclude` and `.rsync-filter`.

## `.rsync-exclude`

This file contains simple exclude rules:

```ini
# ignore all .git/ directories (including nested ones)
.git/

# ignore all *.pyc/*.pyo files (and directories)
*.py[co]

# ignore all build files and directories
build

# ignore the top level .cache/ directory
/.cache/
```

## `.rsync-filter`

This file can contain any filter rules:

```ini
# merge global filter rules
. /home/user/.rsync-global-filter

# recursevely include the Documents/ directory
+ /Documents/***

# except for the nonimportant/ subdirectory
- /Documents/nonimportant/

# recursevely include the Pictures/Photos/Fluffy/ directory;
# any other files and directories in the Pictures/ and the /Pictures/Photos/ directories won't be included;
# all intermediate directories must be explicitly included;
+ /Pictures/
+ /Pictures/Photos/
+ /Pictures/Photos/Fluffy/***

# exclude everything else
- .*
```

`.rsync-filter` is processed _after_ `.rsync-exclude`, therefore it is not possible to whitelist anything that is already blacklisted in `.rsync-exclude`.


[rsync-website]: https://rsync.samba.org/
[rsync-man]: https://download.samba.org/pub/rsync/rsync.1
[rsync-man-filter-rules]: https://download.samba.org/pub/rsync/rsync.1#FILTER_RULES
