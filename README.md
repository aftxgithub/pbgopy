# pbgopy
[![Release](https://img.shields.io/github/release/nakabonne/pbgopy.svg?color=orange&style=flat-square)](https://github.com/nakabonne/pbgopy/releases/latest)
[![go.dev reference](https://img.shields.io/badge/go.dev-reference-007d9c?logo=go&logoColor=white&style=flat-square)](https://pkg.go.dev/mod/github.com/nakabonne/pbgopy?tab=packages)

`pbgopy` acts like [pbcopy/pbpaste](https://www.unix.com/man-page/osx/1/pbcopy/) but for multiple devices. It lets you share data across devices like you copy and paste.

![Demo](assets/demo.gif)

## Installation
Binary releases are available through [here](https://github.com/nakabonne/pbgopy/releases).

#### macOS

```
brew install nakabonne/pbgopy/pbgopy
```

#### RHEL/CentOS

```
rpm -ivh https://github.com/nakabonne/pbgopy/releases/download/v0.3.0/pbgopy_0.3.0_linux_amd64.rpm
```

#### Debian/Ubuntu

```
wget https://github.com/nakabonne/pbgopy/releases/download/v0.3.0/pbgopy_0.3.0_linux_amd64.deb
apt install ./pbgopy_0.3.0_linux_amd64.deb
```

#### Arch Linux

AUR package is available: [pbgopy](https://aur.archlinux.org/packages/pbgopy/)

```
yay pbgopy
```

#### Nix

```
nix-shell -p nixpkgs.pbgopy
```

#### Go

```
go install github.com/nakabonne/pbgopy@latest
```

#### Docker

```
docker run --rm nakabonne/pbgopy pbgopy help
```

## Usage
First up, you start the pbgopy server which works as a shared clipboard for devices. It listens on port 9090 by default.
You must allow access to this port for each device you want to share data with.

```bash
pbgopy serve
```

Populate the address of the host where the above process is running into the `PBGOPY_SERVER` environment variable. Then put the data entered in STDIN into the server with:

```bash
export PBGOPY_SERVER=http://host.xz:9090
pbgopy copy <foo.png
```

Paste it on another device with:

```bash
export PBGOPY_SERVER=http://host.xz:9090
pbgopy paste >foo.png
```


## End-to-end encryption
`pbgopy` comes with a built-in ability to encrypt/decrypt with a variety of keys.

### With symmetric-key:

You can derive the key from password with the `-p` flag, which is provided so that you can encrypt/decrypt without previous setting.
```bash
pbgopy copy -p your-password <plaintext.txt
```

```bash
pbgopy paste -p your-password
```

Be aware that this way cannot prevent a dictionary attack.

For more safety, it is highly recommended to use a 32-bytes symmetric key generated by other methods.
The `-k` flag or the `PBGOPY_SYMMETRIC_KEY_FILE` environment variable is available to indicate the path to key file.

```bash
pbgopy copy -k /path/to/pbgopy.key <plaintext.txt
```

### With public/private key-pair:
`pbgopy` can also encrypt using hybrid cryptosystem. If you have already exchanged public keys between devices you want to share data with, this is the way to go.

```bash
pbgopy copy --public-key-file /path/to/public.key <plaintext.txt
```

```bash
pbgopy paste --private-key-file /path/to/private.key <plaintext.txt
```

Note that you can only use an RSA key in PEM or DER format.

#### Via GPG
You manage your keyring in GPG? The `--gpg-user-id` (`-u`) flag is for you!
Suppose you want to encrypt with a public key whose user id is `alice`:

```bash
pbgopy copy -u alice <plaintext.txt
```

Then you decrypt it with the private key by specifying the user id on another device:

```bash
pbgopy paste -u alice
```

There are a couple of ways to specify a user ID. Visit [here](https://www.gnupg.org/documentation/manuals/gnupg/Specify-a-User-ID.html) to see the entire list.

## TTL
If you don't want more data to be cached on the server than necessary, use the `--ttl` flag to set TTL for the cache.
Give `0s` for disabling it. Default is `24h`.

```bash
pbgopy serve --ttl 10m
```

## Authentication
HTTP Basic Authentication is available with `-a` flag.

```bash
pbgopy serve -a user:pass
```

```bash
pbgopy copy -a user:pass <foo.png
```

```bash
pbgopy paste -a user:pass >foo.png
```

## From clipboard on your OS
You can put the data stored at the clipboard on your OS into pbgopy server.

```bash
pbgopy copy -c
```

## Command-line options

#### Copy
```
pbgopy copy -h
Copy from stdin

Usage:
  pbgopy copy [flags]

Examples:
  export PBGOPY_SERVER=http://host.xz:9090
  echo hello | pbgopy copy

Flags:
  -a, --basic-auth string           Basic authentication, username:password
  -c, --from-clipboard              Put the data stored at local clipboard into pbgopy server
      --gpg-path string             Path to gpg executable (default "gpg")
  -u, --gpg-user-id string          GPG user id associated with public-key to be used for encryption
  -h, --help                        help for copy
      --max-size string             Max data size with unit (default "500mb")
  -p, --password string             Password to derive the symmetric-key to be used for encryption
  -K, --public-key-file string      Path to an RSA public-key file to be used for encryption; Must be in PEM or DER format
  -k, --symmetric-key-file string   Path to symmetric-key file to be used for encryption
      --timeout duration            Time limit for requests (default 5s)
```

#### Paste
```
pbgopy paste -h
Paste to stdout

Usage:
  pbgopy paste [flags]

Examples:
  export PBGOPY_SERVER=http://host.xz:9090
  pbgopy paste >hello.txt

Flags:
  -a, --basic-auth string                  Basic authentication, username:password
      --gpg-path string                    Path to gpg executable (default "gpg")
  -u, --gpg-user-id string                 GPG user id associated with private-key to be used for decryption
  -h, --help                               help for paste
      --max-size string                    Max data size with unit (default "500mb")
  -p, --password string                    Password to derive the symmetric-key to be used for decryption
  -K, --private-key-file string            Path to an RSA private-key file to be used for decryption; Must be in PEM or DER format
      --private-key-password-file string   Path to password file to decrypt the encrypted private key
  -k, --symmetric-key-file string          Path to symmetric-key file to be used for decryption
      --timeout duration                   Time limit for requests (default 5s)
```

#### Serve
```
pbgopy serve -h
Start the server that acts like a clipboard

Usage:
  pbgopy serve [flags]

Examples:
pbgopy serve --port=9090 --ttl=10m

Flags:
  -a, --basic-auth string   Basic authentication, username:password
  -h, --help                help for serve
  -p, --port int            The port the server listens on (default 9090)
      --ttl duration        The time that the contents is stored. Give 0s for disabling TTL (default 24h0m0s)
```

## Inspired By
- [nwtgck/piping-server](https://github.com/nwtgck/piping-server)
- [bradwood/glsnip](https://github.com/bradwood/glsnip)
