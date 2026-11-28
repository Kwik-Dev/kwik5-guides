# kwik5-guides

## What is here

| path | what |
| --- | --- |
| `guides/` | the Hugo project: the landing page at `/`, the manual mounted at `/guides/` |
| `guides/data/en/` | the landing copy that is data, not markup — features, pricing, FAQ, OSS list |
| `docs/guides/` | the manual pages, mounted into `guides/` (never copied) |
| `docs/guides/README.md` | what the manual is for, and its conventions |

```bash
cd guides && hugo server      # → http://localhost:1313/
cd guides && hugo             # → guides/public/
```

No theme, no Hugo module, no network at build time.

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md) and [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md).
Questions about *using* Kwik belong in the manual: https://github.com/Kwik-Dev/kwik5-guides. A vulnerability goes
to [`SECURITY.md`](SECURITY.md), never to a public issue.

## Licence

MIT — see [`LICENSE`](LICENSE). Everything the customer runs is free and open source (D2). The paid
product is **Kwik Cloud**: GPU time and the credit ledger, the one part a laptop cannot fake (D3).
