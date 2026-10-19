# we-have-a-problem.com

Disclosure page for the compromise of eight eMudhra intermediate CA
private keys. It lets visitors check whether their browser still trusts
the compromised TLS intermediates, and links to the list of affected
certificates and a proof of key control.

## How it works

For every host in the `HOSTS` list at the bottom of `index.html`, the
page issues `fetch("https://<host>/")` with a timeout. A `2xx` response
is shown as "Connected"; a failed handshake, non-2xx status or timeout
is shown as "Failed". Browsers do not expose the reason a connection
failed, so "Failed" means "could not connect over HTTPS", not
specifically "certificate rejected".

## Files

- `affected-certificates.json` lists the eight compromised intermediates
  (original and Sectigo cross-sign) with SHA-256 fingerprints, SPKI
  hashes, serials and CCADB IDs, plus the three roots they chain to.
- `proof.zip` (and the `proof/` directory it is built from) holds, for each intermediate, the certificate
  (`NAME.crt.pem`), its public key (`NAME.pub.pem`) and a DER-encoded
  ECDSA P-384 / SHA-384 signature (`NAME.sig`) over `proof/message.txt`.
  To verify:

      cd proof
      openssl x509 -in NAME.crt.pem -noout -pubkey | diff - NAME.pub.pem
      openssl dgst -sha384 -verify NAME.pub.pem -signature NAME.sig message.txt

## Requirements for the test hosts

Each host must answer `GET /` with a `2xx` status and the header

    Access-Control-Allow-Origin: *

so the page can read the response.

## Deploying

The site is plain HTML with no build step. Enable GitHub Pages for the
repository and point it at the branch root. `.nojekyll` is present so
Pages serves the files as-is, and `CNAME` sets the custom domain
`emudhra.we-have-a-problem.com`. That hostname needs a DNS CNAME record
pointing at `<user>.github.io`; the `*-emudhra.we-have-a-problem.com`
test hosts are served by your own servers, not by Pages.
