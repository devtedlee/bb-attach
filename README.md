# bb-attach

Upload images to **Bitbucket Cloud** pull requests and comments from the command line, and get a URL you can drop straight into Markdown.

The Bitbucket equivalent of [`gh-attach`](https://github.com/sudosubin/gh-attach).

## Why

Bitbucket Cloud's public REST API (`api.bitbucket.org/2.0`) has **no PR-scoped attachment endpoint**. When you paste an image into a PR description or comment in the web UI, it goes to an internal, undocumented endpoint authenticated by your **browser session** — not by an API token.

`bb-attach` does exactly what the web UI does: it reads your browser's Bitbucket session cookies and calls that same endpoint, then prints the resulting image URL.

## Install

```bash
pip3 install --user browser-cookie3
curl -o ~/.local/bin/bb-attach https://raw.githubusercontent.com/devtedlee/bb-attach/main/bb-attach
chmod +x ~/.local/bin/bb-attach   # make sure ~/.local/bin is on your PATH
```

Requires Python 3.7+ and a browser (Chrome/Firefox/Safari/Edge/Brave) logged in to bitbucket.org.

## Usage

```bash
bb-attach shot.png                       # print the URL (repo inferred from git origin)
bb-attach -m shot.png                    # ![name](url) Markdown, ready to paste
bb-attach --json a.png b.png             # JSON array
bb-attach --repo myworkspace/myrepo x.png   # specify repo explicitly
bb-attach --browser firefox x.png        # cookie source browser (default: chrome)
```

Paste the printed URL (or the `-m` Markdown) into any PR description or comment — it renders inline, exactly like a web-UI upload.

## How it works

1. Resolve `workspace/repo` from `--repo` or the `git origin` remote.
2. Read `bitbucket.org` session cookies from your browser via [`browser-cookie3`](https://github.com/borisbabic/browser_cookie3).
3. `POST` the file (multipart, field `file`) to `https://bitbucket.org/xhr/{workspace}/{repo}/image-upload/` with the `X-CSRFToken` header set from the `csrftoken` cookie.
4. Print the `href` from the JSON response.

## Privacy & caveats

- **Reads your browser cookies.** Cookies are held in memory and sent only to `bitbucket.org` over HTTPS — never written to disk or logged. This is the same mechanism `gh-attach` uses for GitHub.
- **Uses an undocumented endpoint.** If Atlassian changes it, this breaks. The tool fails loudly (non-zero exit) rather than emitting a wrong URL if the response shape changes.
- Uploaded images inherit the **repository's** access level (private repo → viewers need repo read access).

## License

MIT
