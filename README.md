# Mail Check

A small, dependency-free tool that audits a domain's email security using public DNS records and published policy files. It never connects to the target's mail servers.

Everything lives in one file, `server.js`. It runs on Node.js 18 or newer and needs no `npm install`.

## What it checks

| Check | What it looks at |
| --- | --- |
| SPF | Single record, strictness of the `all` ending, the 10 DNS lookup limit, dead `include:` targets |
| DKIM | Public keys at common selectors, plus one you supply; revoked or short keys |
| DMARC | Policy (`none`, `quarantine`, `reject`), `pct`, reporting address |
| MX | Records exist, hosts resolve, redundancy, null MX |
| MTA-STS | TXT record, the policy file over HTTPS, `enforce` vs `testing` |
| TLS-RPT | TLS failure reporting record |

The result is a score out of 100, a letter grade, and for each check a plain-English explanation, the raw record, and a suggested fix.

Scoring: each problem costs 20 points, each warning 8. Notes cost nothing.

## Run it

```bash
node server.js
```

Then open http://127.0.0.1:3000.

Or query the JSON API directly:

```bash
curl "http://127.0.0.1:3000/api/check?domain=example.com&selector=google"
```

## Configuration

Set these at the top of `server.js`:

- `COPYRIGHT`: the line shown in the page footer
- `SITE_NAME`: the title and heading
- `RATE_LIMIT`: checks per minute per IP

Environment variables:

| Variable | Default | Purpose |
| --- | --- | --- |
| `PORT` | `3000` | Port to listen on |
| `HOST` | `127.0.0.1` | Interface to bind to |
| `TRUST_PROXY` | unset | Set to `1` behind a reverse proxy so the rate limiter reads `X-Forwarded-For` |

## Hosting on a subdomain

Run it as a service and put a reverse proxy in front for TLS. A minimal nginx block:

```nginx
server {
    server_name check.example.com;
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header X-Forwarded-For $remote_addr;
    }
}
```

Start the app with `TRUST_PROXY=1`. Only enable that when the proxy overwrites the header, otherwise clients can spoof it.

## What it does and does not do

It is passive. It only makes:

- DNS queries to your resolver
- One HTTPS request for `https://mta-sts.<domain>/.well-known/mta-sts.txt`

The MTA-STS fetch refuses redirects and non-public addresses so the tool cannot be pointed at internal hosts.

It does not open SMTP connections, test for open relays, enumerate mailboxes, scan ports, or send mail. Contributions that add any of these will not be accepted.

## Limitations

- DKIM selectors cannot be discovered through DNS. If yours is unusual, enter it in the form.
- DNSSEC, DANE, BIMI and blocklist checks are not included yet.
- SPF lookup counting treats each `mx` and `a` as one lookup and does not expand their results, which matches how the limit is usually counted but may differ slightly from some validators.
- Results depend on the resolver your server uses.

## Contributing

Issues and pull requests are welcome. Keep it to one file with no runtime dependencies, and keep checks passive.
please suggest if you have any idea to add on it.
## License

MIT. See `LICENSE`.
