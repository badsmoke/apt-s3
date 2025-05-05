# apt-s3
![repo-checks](https://github.com/badsmoke/apt-s3/workflows/repo-checks/badge.svg)

`apt-s3` is an [APT Method Interface](http://www.fifi.org/doc/libapt-pkg-doc/method.html/) written in Go to use a private S3 bucket as an `apt` repository on Debian based systems. Similar projects exist, but they all have their caveats:
  * Many are completely unmaintained
  * Most require `python` and some require additional `pip` packages
  * Some only use the default AWS authentication methods
    * This means any application specific credentials in a Docker container must also have access to the S3 bucket or `apt` breaks entirely
  * Most set the region globally so they only support a single S3 region at a time
  * Some place the API keys in the S3 URI
    * This means they are leaked every time `apt-get update` or `apt-get install` is run
  * Some do not use the AWS SDK
  * None of them expose an interactive component for downloading files

## Installation

The only requirement for `apt-s3` is the `ca-certificates` package and its dependencies.

Installation is as easy as downloading the binary or deb package from our [releases](https://github.com/badsmoke/apt-s3/releases) page.

### Package Installation

Download the package and install it with `dpkg -i /path/to/package.deb`. If you see the error message below simply run `apt-get install -f` to fix it.
```
dpkg: dependency problems prevent configuration of apt-s3:
 apt-s3 depends on ca-certificates; however:
  Package ca-certificates is not installed.
```

### Binary Installation

Download the binary and move it to `/usr/lib/apt/methods/s3`.

## Usage

Simply create an apt list file in the proper format to start using `apt-s3` with apt.
```bash
export BUCKET_NAME=my-s3-bucket
export BUCKET_REGION=us-east-1

echo "deb s3://${BUCKET_NAME}.s3-${BUCKET_REGION}.amazonaws.com/ stable main" > /etc/apt/sources.list.d/s3bucket.list
```

### Credentials File

`/etc/apt/s3creds` is checked before using the default AWS credential methods. The file has a format similar to `~/.aws/credentials`, but profiles are ignored.

```
aws_access_key_id     = foo
aws_secret_access_key = foobar123
aws_session_token     = not-normally-needed
```

### Interactive Usage

To download a file using `apt-s3` simply use the `-download` flag. Run `apt-s3 -help` for usage info.

```bash
export BUCKET_NAME=my-s3-bucket
export BUCKET_REGION=us-east-1

apt-s3 -download s3:/${BUCKET_NAME}.s3-${BUCKET_REGION}.amazonaws.com/file -path /tmp/file
```

## Building

Use the Makefile to build the binary and .deb package (requires [nfpm](https://github.com/goreleaser/nfpm) to be installed and in the `$PATH`).

```bash
$ make
```

## Releasing a New Version

To release a new version you will need a few things:

1. Write access to this repo
2. [A personal access token](https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line)
3. [nfpm](https://github.com/goreleaser/nfpm) installed and in the `$PATH`

Once everything is set up follow these steps to create a release and upload assets:

```bash
export GITHUB_TOKEN=<your personal access token>
# bumping the version programatically can be easily done with `awk`
export VERSION=$(git describe --tags | awk -F. -v OFS=. '{ $3++ } 1') # use $1 for major/$2 for minor/$3 for patch
make VERSION=$VERSION release
```

## Contributing

Improvements are always welcome. Please follow these steps to contribute:

1. Fork repo
2. Submit a Pull Request with a detailed explanation of changes
3. Receive a :+1: from a core team member
4. Core team will merge your changes

## License

Use of this software is subject to important terms and conditions as set forth in the [LICENSE](LICENSE) file.
