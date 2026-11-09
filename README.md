# BandcampSync

BandcampSync is a tool which synchronises media purchased on a [Bandcamp](https://bandcamp.com/)
account with a local directory. It is available as a Python module, command line
script, or packed in a Docker container.

You may use this to download media you have purchased from Bandcamp to a
local media server, such as Plex or Jellyfin. Media is saved according to
[Lidarr's recommended folder structure](https://wiki.servarr.com/en/lidarr/quick-start-guide#folder-structure). Any of Bandcamp's high quality
download options are available with BandcampSync (defaulting to FLAC).

## How it works

When called, BandcampSync will:

1. Authenticate with bandcamp.com using your exported session cookies
2. Scan your local media directory for existing downloaded items
3. Index your purchased items in your Bandcamp collection
4. Download items not already downloaded from your collection
5. Unzip the archive (if required) and move the contents to your specified local
   media directory

### Media directory format

Downloaded media is saved to a local media directory (default: `/downloads`) in
the format: `artist_name`/`item_title`:

```text
downloads/
└─ Artist Name 1/
   ├─ Album Name 1/
   │  ├─ bandcamp_item_id.txt
   │  ├─ cover.jpg
   │  ├─ Track Name 1.flac
   │  ├─ Track Name 2.flac
   │  └─ Track Name 3.flac
   ├─ Album Name 2/
   │  └─ ...
   └─ Album Name 3/
      └─ ...
```

This directory format is fixed according to [Lidarr's recommended folder structure](https://wiki.servarr.com/en/lidarr/quick-start-guide#folder-structure),
although the names of the directories can be changed once items are downloaded.

BandcampSync saves a `bandcamp_item_id.txt` file to each item directory so that
it can track which items are downloaded independent of directory renaming.
Deleting this file will cause BandcampSync to re-download the item the next time
it is run.

## Installation

BandcampSync is available as a Python module, a standalone CLI script, or as a
minimal Docker container (batteries-included image provided).

### Python

The `bandcampsync` module is pure Python and depends only on the `requests` and
`beautifulsoup4` libraries. Any modern version of Python3 will be compatible.

You can install `bandcampsync` via pip:

```bash
$ pip install bandcampsync
```

<!-- TODO: add details on CLI script setup -->

### Docker

The Docker image contains the `bandcampsync` Python module, as well as a helper
script that runs the `bandcampsync` on a daily timer.

You can pull and run the image with the following commands:

```bash
# Pull image
$ docker pull ghcr.io/meeb/bandcampsync:latest

# Start the container using your user ID and group ID
$ docker run \
  -d \
  --name bandcampsync \
  -e TZ=Europe/London \
  -e PUID=1000 \
  -e PGID=1000 \
  -e RUN_DAILY_AT=3 \
  -v /some/directory/bandcampsync-config:/config \
  -v /some/directory/bandcampsync-media:/downloads \
  ghcr.io/meeb/bandcampsync:latest

```

Or an example Docker Compose entry:

```yaml
version: "3.7"
services:
  bandcampsync:
    image: ghcr.io/meeb/bandcampsync:latest
    container_name: bandcampsync
    restart: unless-stopped
    environment:
      - TZ=Europe/London
      - PUID=1000
      - PGID=1000
      - RUN_DAILY_AT=0
    volumes:
      - /some/directory/bandcampsync-config:/config
      - /some/directory/bandcampsync-media:/downloads
```

#### Configuration

> [!IMPORTANT]
> Remember to provide authenticated session cookies in the

In the above example you would save your cookies data into a file called
`cookies.txt` and save it at `/some/directory/bandcampsync-config/cookies.txt`.
BandcampSync will look for this location when it starts up.

The Docker container can be configured with the following environment variables:

| Variable       | Default value         | Description                                                                                                                                  |
| -------------- | --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| PUID\*         | `1000`                | (don't change unless necessary)                                                                                                              |
| PGID\*         | `1000`                | (don't change unless necessary)                                                                                                              |
| TZ             | `UTC`                 | Time zone identifier of host<br/>(use a "TZ identifier" from [this list](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List)) |
| COOKIES_FILE   | `/config/cookies.txt` | Path to the text file containing your<br/>session cookies                                                                                    |
| DIRECTORY      | `/downloads`          | The                                                                                                                                          |
| FORMAT         | `flac`                |
| IGNORE         | (none)                |
| RUN_DAILY_AT   | `3`                   | The hour at which BandcampSync should automatically run                                                                                                       |
| EXIT_AFTER_RUN | `0`                   |
| TEMP_DIR       | `.`                   | Temporary download files directory                                                                                                           |

\*variable is not optional

| Media sync directory | `/downloads` | `-d`, `--directory` |
| Temporary files directory | `.` | `-t`, `--temp-dir` |
| Path to cookies file | `/config/cookies.txt` | `-c`, `--cookies` |

tz_name = TZ 'UTC'
cookies_path_env = COOKIES_FILE '/config/cookies.txt'
dir_path_env = DIRECTORY '/downloads'
media_format_env = FORMAT 'flac'
ign_patterns = IGNORE ''
run_daily_at_env = RUN_DAILY_AT '3'
exit_after_run_env = EXIT_AFTER_RUN '0'
temp_dir_env = TEMP_DIR ''
cookies_path = cookies_path_env
dir_path = dir_path_env

The `RUN_DAILY_AT` environment variable is the hour the `bandcampsync` script
will run at. In this example, 3am local time. After running the container will
sleep until the following 3am. It will run daily. There is also a randomised
delay added to the hour to not dogpile bandcamp.com with requests on the hour
so the script won't run exactly on the hour.

`RUN_DAILY_AT` should be a number between 0 and 23 (specifying an hour).

`PUID` and `PGID` are the user and group IDs to attempt run the download as.
This sets the UID and GID of the files that are downloaded.

`TEMP_DIR` variable can be set to a directory in the container. If set the
directory is used as the temporary download location.

## Configuration

Before BandcampSync can run, valid session cookies from an authenticated
Bandcamp account must be provided.

There are a few launch options
with sensible defaults,
and

### Launch options

When launching from a which can be set to tweak how the script operates

| Option                    | Default value         | CLI flag            | Docker variable |
| ------------------------- | --------------------- | ------------------- | --------------- |
| Media sync directory      | `/downloads`          | `-d`, `--directory` |
| Temporary files directory | `.`                   | `-t`, `--temp-dir`  |
| Path to cookies file      | `/config/cookies.txt` | `-c`, `--cookies`   |

'-v', '--version', action='store_true',
Displays the bandcampsync version and exits

'-c', '--cookies', required=True,
Path to the cookies file

'-d', '--directory', required=True,
Path to the directory to download media to

'-i', '--ignore', default='',
A space-delimited list of patterns matching artists to bypass

'-f', '--format', default='flac',
Media format to download, defaults to "flac"

'-t', '--temp-dir', default='',
Path to use for temporary downloads

### Obtaining session cookies

The easiest way to get these cookies to go to [bandcamp.com](https://bandcamp.com/)
in your browser and log in with your account.

Next, open the developer tools in your browser (F12 button on most browsers, or
select "developer tools" from the options menu). Go to the "Network" tab

Reload the index page and find the index page request in your network requests
tab of your browser.

![Getting your session cookies](https://github.com/meeb/bandcampsync/blob/main/docs/cookies.jpg?raw=true)

Go to the "Request Headers" section then select and copy
the string after the `Cookie` header. The string should look something like
this:

```text
client_id=00B1F3C8EB48E181A185CCD041E40C0E8F; session=1%0893C88%570EE405455%%8DEC37B5BC393983DB983DD%%BDFD46C3B8A0%%580DA466D5CD; identity=1%HhehuehUFEUiuebn%%2ADB72300DAE573%BEEF389A1B526EA35AC38019FA0A6F%11B4BD5FBC18B83F720; js_logged_in=1; logout=%7B%22username%22%3A%22someuser%22%7D; download_encoding=401; BACKENDID3=some-sever-name
```

Save this string to a file called `cookies.txt`.

You need to save your session ID from cookies manually because Bandcamp has
a captcha on the login form so BandcampSync can't log in with your username
and password for you.

> [!WARNING]
> Keep the `cookies.txt` file safe! Anyone with access to this file
> can log into your Bandcamp account, impersonate you, potentially make purchases
> and generally have total access to your Bandcamp account!

## CLI usage

Once you have the Python `bandcampsync` module installed you can call it with the
`bandcampsync` command:

```bash
$ bandcampsync --cookies cookies.txt --directory /path/to/music
```

or in shorthand:

```bash
$ bandcampsync -c cookies.txt -d /path/to/music
```

You can also use `-t` or `--temp-dir` to set the temporary download directory used. See
`-h` or `--help` for the full list of command line options.

## Formats

By default, BandcampSync will download your music in the `flac` format. You can specify
another format with the `--format` argument. Common Bandcamp download formats are:

| Name            | Description                                                     |
| --------------- | --------------------------------------------------------------- |
| `mp3-v0`        | Variable bitrate MP3. Small file sizes. OK quality.             |
| `mp3-320`       | High quality MP3. Medium file sizes. Good quality.              |
| `flac`          | Losses audio. Large file sizes. Original Quality.               |
| `aac-hi`        | Apple variable bitrate format. Small file sizes. OK quality.    |
| `aiff-lossless` | Uncompressed audio format. Biggest file size. Original quality. |
| `vorbis`        | Open source lossy format. Small file sizes. OK quality.         |
| `alac`          | Apple lossless format. Large file sizes. Original quality.      |
| `wav`           | Uncompressed audio format. Biggest file size. Original quality. |

You can also use `-i` or `--ignore` to bypass artists that have data issues that
your OS can not handle:

```bash
$ bandcampsync --cookies cookies.txt --directory /path/to/music --ignore "badband"
```

Alternatively, items in your Bandcamp collection which have been hidden from public
view will not be downloaded. See the [Bandcamp help center article](https://get.bandcamp.help/hc/en-us/articles/23020724942359-How-do-I-edit-my-Fan-Page)
for how to hide items.

# Contributing

All properly formatted and sensible pull requests, issues and comments are welcome.
