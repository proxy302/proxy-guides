# How do I make git clone and push go through a proxy?

For HTTPS remotes, run `git config --global http.proxy http://USER:PASS@HOST:PORT`, or scope it to one host with `git config --global http.https://github.com.proxy http://USER:PASS@HOST:PORT`. An IP sold as SOCKS5 goes in the same setting as `socks5h://USER:PASS@HOST:PORT`. These settings do not touch SSH remotes (`git@github.com:...`); for those, add a `ProxyCommand` to `~/.ssh/config`, or switch the remote to HTTPS, which is simpler when the proxy needs a password.

*Proxy302 maintains this guide and sells these proxies. The HTTPS commands ran with Git 2.54 against local authenticating HTTP and SOCKS5 test proxies, and the SSH `ProxyCommand` with `nc` ran against a local proxy without a password, in November 2026.*

## HTTPS remotes

**For one command**, without changing any config:

```bash
git -c http.proxy=http://USER:PASS@HOST:PORT ls-remote https://github.com/git/git HEAD
```

If this prints a commit hash, the proxy, the credentials and Git all work together. `ls-remote` downloads almost nothing, so it is a cheap test before a large clone.

**For one host only** (recommended), so your company Git server keeps its normal route:

```bash
git config --global http.https://github.com.proxy http://USER:PASS@HOST:PORT
```

**For everything Git fetches over HTTP(S):**

```bash
git config --global http.proxy http://USER:PASS@HOST:PORT
```

**For one repository**, run the same command inside it without `--global`.

**SOCKS5**: use `socks5h://USER:PASS@HOST:PORT` in any of the commands above. The `h` makes the proxy resolve hostnames.

Git also honours the `https_proxy` and `HTTPS_PROXY` environment variables, through the curl library it uses for HTTP. A value in `http.proxy` overrides them.

### Removing it again

```bash
git config --global --unset http.proxy
git config --global --unset http.https://github.com.proxy
git config --global --get-regexp 'proxy'   # list whatever is left
```

### Keeping the password out of the config file

`~/.gitconfig` is plain text. If you write the proxy as `http://USER@HOST:PORT`, with no password, Git asks for it through its normal credential prompt and can store it with your configured credential helper.

## SSH remotes

`http.proxy` has no effect on `git@host:repo` URLs; SSH opens its own connection. Tell SSH how to reach the proxy in `~/.ssh/config`:

```sshconfig
Host github.com
    ProxyCommand nc -X connect -x HOST:PORT %h %p
```

That works on macOS and on systems with OpenBSD `nc`, but `nc` cannot send a proxy password non-interactively. For a proxy with a username and password, Nmap's `ncat` can:

```sshconfig
Host github.com
    ProxyCommand ncat --proxy HOST:PORT --proxy-type http --proxy-auth USER:PASS %h %p
```

Use `--proxy-type socks5` for a SOCKS5 IP. Test with `ssh -T git@github.com`; a greeting or "Permission denied (publickey)" both prove the connection got through, since the second only means your SSH key was not accepted.

When you would rather not maintain this, change the remote to HTTPS: `git remote set-url origin https://github.com/OWNER/REPO.git`.

## Which setting for which case

| Remote URL | Where the proxy goes | Password supported |
|---|---|---|
| `https://...` | `http.proxy` or `http.<url>.proxy` | Yes, in the URL or via the credential prompt |
| `https://...`, temporary | `git -c http.proxy=...` or `HTTPS_PROXY` | Yes |
| `git@host:...` (SSH) | `ProxyCommand` in `~/.ssh/config` | With `ncat --proxy-auth`; not with `nc` |

## Common errors

| Message | Cause |
|---|---|
| `fatal: unable to access '...': CONNECT tunnel failed, response 407` | Wrong username or password for the proxy |
| `fatal: could not read Password for 'http://USER@HOST:PORT': terminal prompts disabled` | Password left out of the URL in a non-interactive run (CI, IDE); add it or use a credential helper |
| `Failed to connect to HOST port PORT` | Wrong host or port, or the network blocks the proxy port |
| `Unsupported proxy syntax` | `@`, `:` or `#` in the password not percent-encoded |
| `SSL certificate problem: self-signed certificate in certificate chain` | A TLS-inspecting company proxy; see [SSL errors through a proxy](../proxy-ssl-errors/README.md). Do not switch off `http.sslVerify` |
| SSH clone ignores the proxy | `http.proxy` only covers HTTPS remotes; add a `ProxyCommand` |

## FAQ

**Why does the proxy log show a refused attempt before every successful one?** Git's default `http.proxyAuthMethod` is `anyauth`, which first connects without credentials to see what the proxy asks for. Setting `git config --global http.proxyAuthMethod basic` sends the password straight away.

**Does this work in Git for Windows?** Yes. The same `git config` commands apply; Git for Windows ships its own curl with SOCKS5 support.

**Rotating or static for Git?** A clone is a long transfer over one connection, and a server that allowlists your address needs it to stay put. A static residential (ISP) IP fits both.

**Can a proxy get around a Git host's rate limits or blocks?** That is not what it is for. Sign in and stay within the host's terms.

## Where Proxy302 fits

Proxy302 static residential (ISP) IPs come as HTTP or SOCKS5, chosen at purchase, and either goes into `http.proxy`. A static IP costs $0.50 for one day or $7.99 for 30 days, the same in every country. See [pricing](https://proxy302.com/pricing/).

*Facts checked 19 November 2026.*
