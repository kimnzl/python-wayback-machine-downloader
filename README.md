# python wayback machine downloader

[![PyPI](https://img.shields.io/pypi/v/pywaybackup)](https://pypi.org/project/pywaybackup/)
[![PyPI - Downloads](https://img.shields.io/pypi/dm/pywaybackup)](https://pypi.org/project/pywaybackup/)
![Python Version](https://img.shields.io/badge/Python-3.8-blue)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Downloading archived web pages from the [Wayback Machine](https://archive.org/web/).

Internet-archive is a nice source for several OSINT-information. This tool is a work in progress to query and fetch archived web pages.

This tool allows you to download content from the Wayback Machine (archive.org). You can use it to download either the latest version or all versions of web page snapshots within a specified range.

## Installation

### Pip

1. Install the package <br>
   ```pip install pywaybackup```
2. Run the tool <br>
   ```waybackup -h```

### Manual

1. Clone the repository <br>
   ```git clone https://github.com/bitdruid/python-wayback-machine-downloader.git```
2. Install <br>
   ```pip install .```
   - in a virtual env or use `--break-system-package`

## Important notes

- Linux recommended: On Windows machines, the path length is limited. This can only be overcome by editing the registry. Files that exceed the path length will not be downloaded.
- If you query an explicit file (e.g. a query-string `?query=this` or `login.html`), the `--explicit`-argument is recommended as a wildcard query may lead to an empty result.
- The tool uses a sqlite database to handle snapshots. The database will only persist while the download is running.

<br>
<br>

## Arguments

- `-h`, `--help`: Show the help message and exit.
- `-v`, `--version`: Show information about the tool and exit.

### Required

- **`-u`**, **`--url`**:<br>
  The URL of the web page to download. This argument is required.

#### Mode Selection (Choose One)
- **`-a`**, **`--all`**:<br>
  Download snapshots of all timestamps. You will get a folder per timestamp with the files available at that time.
- **`-l`**, **`--last`**:<br>
  Download the last version of each file snapshot. You will get one directory with a rebuild of the page. It contains the last version of each file of your specified `--range`.
- **`-f`**, **`--first`**:<br>
  Download the first version of each file snapshot. You will get one directory with a rebuild of the page. It contains the first version of each file of your specified `--range`.
- **`-s`**, **`--save`**:<br>
  Save a page to the Wayback Machine. (beta)

#### Optional query parameters

- **`-e`**, **`--explicit`**:<br>
  Only download the explicit given URL. No wildcard subdomains or paths. Use e.g. to get root-only snapshots. This is recommended for explicit files like `login.html` or `?query=this`.

- **`--filetype`** `<filetype>`:<br>
  Specify filetypes to download. Default is all filetypes. Separate multiple filetypes with a comma. Example: `--filetype jpg,css,js`. A filter will result in a filtered cdx-file. So if you want to download all files later, you need to query again without the filter. Filetypes are filtered as they are in the snapshot. So if there is no explicit `html` file in the path (common practice) then you cant filter them.

- **`--limit`** `<count>`:<br>
Limits the amount of snapshots to query from the CDX server. If an existing CDX file is injected, the limit will have no effect. So you would need to set `--keep`.

- **Range Selection:**<br>
  Specify the range in years or a specific timestamp either start, end, or both. If you specify the `range` argument, the `start` and `end` arguments will be ignored. Format for timestamps: YYYYMMDDhhmmss. You can only give a year or increase specificity by going through the timestamp starting on the left.<br>
  (year 2019, year 2019, year+month+day 20190101, year+month+day+hour 2019010112)
   - **`-r`**, **`--range`**:<br>
     Specify the range in years for which to search and download snapshots.
   - **`--start`**:<br>
     Timestamp to start searching.
   - **`--end`**:<br>
     Timestamp to end searching.

### Optional

#### Behavior Manipulation

- **`-o`**, **`--output`**:<br>
Defaults to `waybackup_snapshots` in the current directory. The folder where downloaded files will be saved.

<!-- - **`--verbosity`** `<level>`:<br>
Sets verbosity level. Options are `info`and `trace`. Default is `info`. -->

- **`--log`** <!-- `<path>` -->:<br>
Saves a log file into the output-dir. Named as `waybackup_<sanitized_url>.log`.

- **`--progress`**:<br>
Shows a progress bar instead of the default output.

- **`--workers`** `<count>`:<br>
Sets the number of simultaneous download workers. Default is 1, safe range is about 10. Be cautious as too many workers may lead to refused connections from the Wayback Machine.

- **`--no-redirect`**:<br>
Disables following redirects of snapshots. Useful for preventing timestamp-folder mismatches caused by Archive.org redirects.
  
- **`--retry`** `<attempts>`:<br>
Specifies number of retry attempts for failed downloads.

- **`--delay`** `<seconds>`:<br>
Specifies delay between download requests in seconds. Default is no delay (0).

<!-- - **`--convert-links`**:<br>
If set, all links in the downloaded files will be converted to local links. This is useful for offline browsing. The links are converted to the local path structure. Show output with `--verbosity trace`. -->

#### Job Handling:

- **`--reset`**:  
  If set, the job will be reset, and any existing `cdx`, `db`, `csv` files will be **deleted**. This allows you to start the job from scratch without considering previously downloaded data.

- **`--keep`**:  
  If set, all files will be kept after the job is finished. This includes the `cdx` and `db` file. Without this argument, they will be deleted if the job finished successfully.

<br>
<br>

## Usage

### Handling Interrupted Jobs

`pywaybackup` resumes interrupted jobs. The tool automatically continues from where it left off.

- Detects existing `.cdx` and `.db` files in an `output dir` to resume downloading from the last successful point.
- Compares `URL`, `mode`, and `optional query parameters` to ensure automatic resumption.
- Skips previously downloaded files to save time.
> **Note:** Changing URL, mode selection, query parameters or output prevents automatic resumption.

#### Resetting a Job (`--reset`)
- Deletes `.cdx` and `.db` files and restarts the process from scratch.
- Does **not** remove already downloaded files.
- `waybackup -u https://example.com -a --reset`

#### Keeping Job Data (`--keep`)
- Normally, `.cdx` and `.db` files are deleted after a successful job.
- `--keep` preserves them for future re-analysis or extending the query.
- `waybackup -u https://example.com -a --keep`

<br>
<br>

## Examples

1. Download a specific single snapshot of all available files (starting from root):<br>
`waybackup -u https://example.com -a --start 20210101000000 --end 20210101000000`
2. Download a specific single snapshot of all available files (starting from a subdirectory):<br>
`waybackup -u https://example.com/subdir1/subdir2/assets/ -a --start 20210101000000 --end 20210101000000`
3. Download a specific single snapshot of the exact given URL (no subdirs):<br>
`waybackup -u https://example.com -a --start 20210101000000 --end 20210101000000 --explicit`
4. Download all snapshots of all available files in the given range:<br>
`waybackup -u https://example.com -a --start 20210101000000 --end 20231122000000`

<br>
<br>

## Output

### Path Structure

The output path is currently structured as follows by an example for the query:<br>
`http://example.com/subdir1/subdir2/assets/`
<br><br>
For the first and last version (`-f` or `-l`):
- Will only include all files/folders starting from your query-path.
```
your/path/waybackup_snapshots/
└── the_root_of_your_query/ (example.com/)
    └── subdir1/
        └── subdir2/
            └── assets/
                ├── image.jpg
                ├── style.css
                ...
```
For all versions (`-a`):
- Will create a folder named as the root of your query. Inside this folder, you will find all timestamps and per timestamp the path you requested.
```
your/path/waybackup_snapshots/
└── the_root_of_your_query/ (example.com/)
    ├── yyyymmddhhmmss/
    │   ├── subidr1/
    │   │   └── subdir2/
    │   │       └── assets/
    │   │           ├── image.jpg
    │   │           └── style.css
    ├── yyyymmddhhmmss/
    │   ├── subdir1/
    │   │   └── subdir2/
    │   │       └── assets/
    │   │           ├── image.jpg
    │   │           └── style.css
    ...
```

### CSV

Each snapshot is stored with the following keys/values. These are either stored in a sqlite database while the download is running or saved into a CSV file after the download is finished.

For download queries:

```
[
   {
      "file": "/your/path/waybackup_snapshots/example.com/yyyymmddhhmmss/index.html",
      "id": 1,
      "redirect_timestamp": "yyyymmddhhmmss",
      "redirect_url": "http://web.archive.org/web/yyyymmddhhmmssid_/http://example.com/",
      "response": 200,
      "timestamp": "yyyymmddhhmmss",
      "url_archive": "http://web.archive.org/web/yyyymmddhhmmssid_/http://example.com/",
      "url_origin": "http://example.com/"
   },
    ...
]
```

### Debugging

Exceptions will be written into `waybackup_error.log` (each run overwrites the file).

<br>
<br>

## Contributing

I'm always happy for some feature requests to improve the usability of this tool.
Feel free to give suggestions and report issues. Project is still far from being perfect.

> Please PR from dev into dev.
