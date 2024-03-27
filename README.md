# Certificate-Transparency-Logs

This library quickly and reliably downloads entries from [Certificate Transparency](https://certificate.transparency.dev/howctworks/) into a file for further processing. The table at the end of this document provides a list of Certificate Transparency (CT) logs along with their details.

## Installing Libraries

### Install [RUST](https://www.rust-lang.org/learn/get-started):

    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

### Compiling

    git clone https://github.com/PIR-PIXR/Certificate-Transparency-Logs.git
    cd scrape-ct-log
    cargo build --release

#### Help menu
    /PATH/TO/scrape-ct-log/target/release/scrape-ct-log -h

#### Download 2**20 entries from Google's xenon2024
    /PATH/TO/scrape-ct-log/target/release/scrape-ct-log -n 1048576 -o /PATH/TO/scrape-ct-log/xenon2024_log_entries https://ct.googleapis.com/logs/eu1/xenon2024/

#### Control the output format

By default, the output is in JSON format, which is generally understandable and somewhat human-friendly. However, JSON parsing can be CPU-intensive and verbose, especially for representing binary data such as X.509 certificates. To address this, we offer support for outputting the same information in [CBOR](https://cbor.io/), a binary-based format that can be processed more efficiently for our purposes. 

You can utilize the -f or --format option to specify the output format.

    /PATH/TO/scrape-ct-log -f cbor https://ct.googleapis.com/logs/crucible/

## Top-level structure

At the top level, the output will consist of a "map" (object, dictionary, associative array, hash, what-have-you) with the following keys:

* `log_url` (`string`) -- simply the URL that was provided.
    Kept in here just in case you have lots of output files laying around, and would like to know where they all came from.

* `scrape_begin_timestamp` (`integer`) -- the number of milliseconds since the Unix epoch at which the program was started.

* `scrape_end_timestamp` (`integer`) -- the number of milliseconds since the Unix epoch at which the program finished.
    Unsurprisingly, subtracting `scrape_begin_timestamp` from this value will give you a pretty good idea of how long the scrape took.

* `sth` (`<sth>`) -- The Signed Tree Head that was presented by the server when we started the scrape.

* `entries` (`[<entry>]`) The set of entries that were retrieved during the scrape.
    Note that the entries may not be in the order that they are in the log, which is why each `<entry>` has the log's `entry_number` encoded in it.


## `<sth>`

The Signed Tree Head is a Certificate Transparency data structure giving you information about the state of the log at a given time.
Our structure is a straight-up clone of the format described in the spec; for more details on the fields, consult [the RFC](https://datatracker.ietf.org/doc/html/rfc6962#section-3.5).

* `tree_size` (`integer`)

* `timestamp` (`integer`)

* `sha256_root_hash` (`bytes`)

* `tree_head_signature` (`bytes`)


## `<entry>`

This is the meat of the whole endeavour -- the log entries themselves.
Fields that are not relevant to a particular entry will not be present.

* `entry_number` (`integer`) -- where in the log this particular entry was found.

* `timestamp` (`integer`) -- the number of *milliseconds* since the epoch at which the entry was submitted, or attested to, or whatever.

* `certificate` (`bytes`) -- the DER-encoded X.509 certificate that is included in the entry, either the issued certificate or the "poisoned" certificate that stands in for the final certificate, in the case of a precertificate.

* `precertificate` (`<precert>`) -- the precertificate data, if the entry is a precertificate, and the `--include-precert-data` has been specified.

* `chain` ([`bytes`]) -- the set of DER-encoded certificates that were submitted to the log along with the entry.
    Only present if the `--include-chains` option was provided.


## `<precert>`

Precertificates are handled in a slightly weird manner in CT, in that they're included in two different forms in the log entry -- both an actual X.509 certificate with a "poison" extension, as well as the [`tbsCertificate`](https://www.rfc-editor.org/rfc/rfc5280#section-4.1.1.1) of the to-be-issued certificate along with the hash of the issuer key.
The former is included in the main `<entry>` structure, while this structure holds the latter.

* `issuer_key_hash` (`bytes`) -- the SHA-256 hash of the SPKI of the key which was declared to issue the final certificate.

* `tbs_certificate` (`bytes`) -- the DER-encoded `tbsCertificate` structure of the to-be-issued certificate.


| Log Name                   | Log URL                                         | Log State | MMD   | Temporal Interval Start | Temporal Interval End | Log Operator | Contact Info              |
|----------------------------|-------------------------------------------------|-----------|-------|-------------------------|-----------------------|--------------|---------------------------|
| Google ‘Argon2023’ log     | [https://ct.googleapis.com/logs/argon2023/](https://ct.googleapis.com/logs/argon2023/) | Usable    | 86400 | 2023-01-01T00:00:00Z    | 2024-01-01T00:00:00Z  | Google       | google-ct-logs@googlegroups.com |
| Google ‘Argon2024’ log     | [https://ct.googleapis.com/logs/us1/argon2024/](https://ct.googleapis.com/logs/us1/argon2024/) | Usable    | 86400 | 2024-01-01T00:00:00Z    | 2025-01-01T00:00:00Z  | Google       | google-ct-logs@googlegroups.com |
| Google ‘Xenon2023’ log     | [https://ct.googleapis.com/logs/xenon2023/](https://ct.googleapis.com/logs/xenon2023/) | Usable    | 86400 | 2023-01-01T00:00:00Z    | 2024-01-01T00:00:00Z  | Google       | google-ct-logs@googlegroups.com |
| Google ‘Xenon2024’ log     | [https://ct.googleapis.com/logs/eu1/xenon2024/](https://ct.googleapis.com/logs/eu1/xenon2024/) | Usable    | 86400 | 2024-01-01T00:00:00Z    | 2025-01-01T00:00:00Z  | Google       | google-ct-logs@googlegroups.com |
| Cloudflare ‘Nimbus2023’ Log | [https://ct.cloudflare.com/logs/nimbus2023/](https://ct.cloudflare.com/logs/nimbus2023/) | Usable    | 86400 | 2023-01-01T00:00:00Z    | 2024-01-01T00:00:00Z  | Cloudflare   | ct-logs@cloudflare.com     |
| Cloudflare ‘Nimbus2024’ Log | [https://ct.cloudflare.com/logs/nimbus2024/](https://ct.cloudflare.com/logs/nimbus2024/) | Usable    | 86400 | 2024-01-01T00:00:00Z    | 2025-01-01T00:00:00Z  | Cloudflare   | ct-logs@cloudflare.com     |
| DigiCert Yeti2024 Log      | [https://yeti2024.ct.digicert.com/log/](https://yeti2024.ct.digicert.com/log/)     | Usable    | 86400 | 2024-01-01T00:00:00Z    | 2025-01-01T00:00:00Z  | DigiCert     | ctops@digicert.com         |
| DigiCert Yeti2025 Log      | [https://yeti2025.ct.digicert.com/log/](https://yeti2025.ct.digicert.com/log/)     | Usable    | 86400 | 2025-01-01T00:00:00Z    | 2026-01-01T00:00:00Z  | DigiCert     | ctops@digicert.com         |
| DigiCert Nessie2023 Log    | [https://nessie2023.ct.digicert.com/log/](https://nessie2023.ct.digicert.com/log/)   | Usable    | 86400 | 2023-01-01T00:00:00Z    | 2024-01-01T00:00:00Z  | DigiCert     | ctops@digicert.com         |
| DigiCert Nessie2024 Log    | [https://nessie2024.ct.digicert.com/log/](https://nessie2024.ct.digicert.com/log/)   | Usable    | 86400 | 2024-01-01T00:00:00Z    | 2025-01-01T00:00:00Z  | DigiCert     | ctops@digicert.com         |
| DigiCert Nessie2025 Log    | [https://nessie2025.ct.digicert.com/log/](https://nessie2025.ct.digicert.com/log/)   | Usable    | 86400 | 2025-01-01T00:00:00Z    | 2026-01-01T00:00:00Z  | DigiCert     | ctops@digicert.com         |
| Sectigo ‘Sabre’ CT log     | [https://sabre.ct.comodo.com/](https://sabre.ct.comodo.com/)                     | Usable    | 86400 |                           |                       | Sectigo      | ctops@sectigo.com          |
| Sectigo ‘Mammoth’ CT log   | [https://mammoth.ct.comodo.com/](https://mammoth.ct.comodo.com/)                 | Retired   | 86400 |                           | 2023-01-15 00:00:00Z  | Sectigo      | ctops@sectigo.com          |
| Let’s Encrypt ‘Oak2023’ log| [https://oak.ct.letsencrypt.org/2023/](https://oak.ct.letsencrypt.org/2023/)     | Usable    | 86400 | 2023-01-01T00:00:00Z    | 2024-01-07T00:00:00Z  | Let’s Encrypt | sre@letsencrypt.org        |
| Let’s Encrypt ‘Oak2024H1’ log | [https://oak.ct.letsencrypt.org/2024h1/](https://oak.ct.letsencrypt.org/2024h1/) | Usable    | 86400 | 2023-12-20T00:00:00Z    | 2024-07-20T00:00:00Z  | Let’s Encrypt | sre@letsencrypt.org        |
| Let’s Encrypt ‘Oak2024H2’ log | [https://oak.ct.letsencrypt.org/2024h2/](https://oak.ct.letsencrypt.org/2024h2/) | Usable    | 86400 | 202

## Contributors
 - [Matt Palmer](https://github.com/mpalmer)
 - [Quang Cao](https://github.com/cnquang)
