# Coin-Op

Pay-per-use access to servers, settled on the Xelis network. A Coin-Op server
listens on nothing: clients ask for access by paying, on chain, and the server
answers from its own node.

This repository holds the releases: built programs, their checksums, and the
proofs that they are the ones Coin-Op published. The source is not public.

> **Alpha, on Xelis testnet.** Releases are for trying Coin-Op out with testnet
> XEL, which has no value. The server install is a trial build: SSH stays
> reachable, so the machine is not sealed. Keep your provider's console as the
> way back in.

## Releases

See [Releases](https://github.com/inviti8/coin-op/releases). Each one holds:

| File | What it is |
| --- | --- |
| `coin-op-windows-x64.zip` | the Coin-Op window for Windows, with the Xelis wallet and node it runs |
| `install.sh` | the server install |
| the `*.ext4.gz` images, `host-bin.tar.gz`, `deploy.tar.gz`, `images.sha256` | what `install.sh` puts on a server |
| `THIRD_PARTY_NOTICES` | the licenses of the third-party software in the release |
| `SHA256SUMS` | the SHA-256 of every file above |
| `SHA256SUMS.sig` | the Coin-Op release key's signature over `SHA256SUMS` |
| `SHA256SUMS.authen` | an [Authen](https://authen.hvym.link) notarization of `SHA256SUMS` |

## The window (Windows)

1. Download `coin-op-windows-x64.zip` from a release and unzip it. Keep the
   three programs together.
2. Run `tw-ui.exe`. It isn't code-signed yet, so Windows shows "Windows
   protected your PC": choose *More info*, then *Run anyway*.
3. The window runs its own Xelis node, which catches up with the network in a
   few minutes; the address bar says how far it has come.

Its files are in `%LOCALAPPDATA%\Tokenwall`.

## A server (Ubuntu or Debian, x86_64, with `/dev/kvm`)

As root, with `TAG` the release (e.g. `v0.1.0-alpha.1`):

```sh
curl -fsSLO https://github.com/inviti8/coin-op/releases/download/TAG/install.sh
sha256sum install.sh        # compare with install.sh's line in the release's SHA256SUMS
sh install.sh TAG                 # a new server
sh install.sh TAG SERVER-LINK     # a reinstall: tokenwall:testnet/<admin contract>
```

The install prints the server's boot address. Fund it from the window: *My
servers*, *+ New server* (or, for a reinstall, the server's tab, *Reinstall*).
The server sets itself up, announces itself, and waits for you to approve it
in the window.

A VPS needs nested virtualization for `/dev/kvm`. The server's VMs use about
4 vCPU and 3 GiB of memory, and 40 GiB of disk for the chain.

## How a release is checked

`install.sh` carries two public keys and checks a release against them before
anything runs:

| Key | Pinned in `install.sh` as |
| --- | --- |
| the Coin-Op release key | `RELEASE_KEY` `28e2cf16a77d4a6593b2189a74302181d3f4f6762260ca8770b56a339099d3e6` |
| Authen's notary key | `AUTHEN_KEY` `fd1d17da1fcc56b4fc6e772e49d45cc79ae74db0602c57c10761e398ba156574` |

1. `SHA256SUMS.sig` must be the release key's signature of `SHA256SUMS`.
2. `SHA256SUMS.authen` must be Authen's signature of a statement that it saw
   bytes with the SHA-256 of `SHA256SUMS`, and when.
3. Every file the server needs must match its line in `SHA256SUMS`.

Any failure stops the install. The signature says Coin-Op published these
files; the notarization, from a separate notary, says when they existed. So a
release can't be changed after it is published without failing both, and a
stolen release key can't make a release that appears to be older than the
theft.

The window's zip is listed in the same `SHA256SUMS`, so the same chain covers
it: check its SHA-256 (`Get-FileHash coin-op-windows-x64.zip`) against its
line.
