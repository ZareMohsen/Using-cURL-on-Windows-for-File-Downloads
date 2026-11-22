# Using cURL on Windows for File Downloads

A practical reference for using `curl.exe` in Windows PowerShell. It
covers redirects, resuming interrupted downloads, progress display,
proxy configuration, creating output directories, and downloading
multiple files concurrently.

> **PowerShell note:** Use `curl.exe` to run the actual cURL executable.
> In Windows PowerShell 5.1, `curl` may be an alias for
> `Invoke-WebRequest`.

## Prerequisites

-   cURL for Windows installed
-   An internet connection or a working proxy
-   Example proxy used in this guide: `127.0.0.1:10808`

Check the installed version:

``` powershell
curl.exe --version
```

## Options covered in this guide

  -----------------------------------------------------------------------
  Option                              Purpose
  ----------------------------------- -----------------------------------
  `-O`                                Save using the filename from the
                                      URL path

  `-L` / `--location`                 Follow HTTP redirects

  `-C -` / `--continue-at -`          Attempt to resume an interrupted
                                      download

  `-#` / `--progress-bar`             Display a progress bar

  `-v` / `--verbose`                  Show detailed connection and
                                      request/response information

  `--create-dirs`                     Create the directories needed for
                                      the output path

  `--proxy` / `-x`                    Configure a proxy

  `--parallel`                        Run multiple transfers concurrently

  `--parallel-max 4`                  Allow up to four concurrent
                                      transfers

  `-f` / `--fail`                     Treat HTTP error responses, such as
                                      404 or 500, as failures

  `--show-error`                      Display error messages, especially
                                      when used with silent mode

  `--retry 5`                         Retry transient failures up to five
                                      times

  `--retry-delay 3`                   Wait three seconds between retries

  `--retry-max-time 900`              Limit the retry period to 900
                                      seconds

  `--connect-timeout 30`              Allow up to 30 seconds to establish
                                      a connection
  -----------------------------------------------------------------------

## Configure a proxy

### HTTP proxy

``` powershell
--proxy http://127.0.0.1:10808
```

### SOCKS5 proxy with remote DNS resolution

``` powershell
--proxy socks5h://127.0.0.1:10808
```

With `socks5h`, the hostname is resolved by the proxy rather than
locally. This is useful when DNS requests should also go through the
proxy.

## Download using the filename from the URL

Use `-O` to save the file using the final path component of the URL:

``` powershell
curl.exe -fL `
    --show-error `
    --progress-bar `
    --proxy socks5h://127.0.0.1:10808 `
    -O `
    "https://example.com/files/archive.zip"
```

In this example, the file is saved as `archive.zip`.

> `-O` gets the filename from the URL path. If the server provides the
> filename only through the `Content-Disposition` response header, use
> `-OJ` instead.

## Resume an interrupted download

``` powershell
curl.exe -fL `
    --show-error `
    --progress-bar `
    --continue-at - `
    --proxy socks5h://127.0.0.1:10808 `
    -O `
    "https://example.com/files/archive.zip"
```

`--continue-at -` tells cURL to inspect the local file size and attempt
to resume from that byte position.

**Limitation:** Resuming depends on the server supporting HTTP Range
requests and the remote file remaining unchanged. If the server does not
support this feature, the download may not resume.

## Create output directories automatically

Use `--create-dirs` with `-o` when specifying an output path:

``` powershell
curl.exe -fL `
    --show-error `
    --progress-bar `
    --create-dirs `
    --proxy socks5h://127.0.0.1:10808 `
    -o "$env:USERPROFILE\Downloads\Tools\archive.zip" `
    "https://example.com/files/archive.zip"
```

If the `Tools` directory does not exist, cURL creates it.

## Recommended command for everyday downloads

This example follows redirects, displays a progress bar, retries
transient failures, uses the SOCKS5 proxy, and saves to a specified
output path:

``` powershell
$url = "https://example.com/files/archive.zip"
$output = "$env:USERPROFILE\Downloads\archive.zip"

curl.exe -fL `
    --show-error `
    --progress-bar `
    --retry 5 `
    --retry-delay 3 `
    --retry-max-time 900 `
    --connect-timeout 30 `
    --proxy socks5h://127.0.0.1:10808 `
    --create-dirs `
    --output $output `
    $url
```

To attempt to resume the download after an interruption, add:

``` powershell
--continue-at -
```

This example uses `--output` so that the filename and destination are
explicit. To use the filename from the URL instead, replace
`--output $output` with `-O`.

## Show connection details for troubleshooting

Add `-v` to inspect connection details, TLS negotiation, proxy activity,
and request/response headers:

``` powershell
curl.exe -fL `
    --verbose `
    --proxy socks5h://127.0.0.1:10808 `
    -O `
    "https://example.com/files/archive.zip"
```

Verbose output can contain sensitive information, such as authentication
headers or cookies. Review and redact it before sharing logs or
publishing them on GitHub.

## Download multiple files concurrently

``` powershell
curl.exe -fL `
    --parallel `
    --parallel-max 4 `
    --proxy socks5h://127.0.0.1:10808 `
    -O "https://example.com/files/file1.zip" `
    -O "https://example.com/files/file2.zip" `
    -O "https://example.com/files/file3.zip"
```

This runs up to four independent transfers at the same time.
`--parallel` does **not** split one file into multiple parts; it
downloads multiple URLs concurrently.

## Verify a downloaded file

### Check file information

``` powershell
Get-Item "$env:USERPROFILE\Downloads\archive.zip" |
    Select-Object Name, Length, LastWriteTime
```

### Calculate the SHA-256 hash

``` powershell
Get-FileHash "$env:USERPROFILE\Downloads\archive.zip" -Algorithm SHA256
```

If the publisher provides an official checksum, compare it with the
calculated value.

## Important notes

-   Use `-O` to take the filename from the URL path; use `-o` to specify
    the output filename and path yourself.
-   Use `-L` to follow redirects.
-   Use `-C -` to attempt to resume an interrupted download. The server
    must support HTTP Range requests.
-   `--parallel` downloads multiple files concurrently; it does not
    split a single file into parts.
-   `-v` is useful for troubleshooting, but its output may expose
    sensitive information.
-   A successful download does not guarantee that a file is authentic or
    safe. Verify its checksum or digital signature when possible.

## Official resources

-   [cURL website](https://curl.se/)
-   [cURL command-line options](https://curl.se/docs/manpage.html)
-   [Everything curl](https://everything.curl.dev/)
