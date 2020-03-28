# scalpel

[![Build Status](https://ci.spearow.io/api/v1/teams/collab/pipelines/scalpel/jobs/master-validate/badge)](https://ci.spearow.io/teams/collab/pipelines/scalpel) [![Crates.io](https://img.shields.io/crates/v/scalpel-bin.svg)](https://crates.io/crates/scalpel-bin) [![License](https://img.shields.io/crates/l/scalpel-bin.svg)](#license)


A scalpel and stitch tool for binaries. Maybe also a signing tool, maybe.

### Snip around, stitch up or graft binaries

This is mostly used for the case where parts of the binary need to be extracted or replaced.

#### Use Cases

* stance firmware into pieces from an all-in-one blob

    ```bash
    scalpel stance --range 0..4Ki --output bootloader.bin firmware.bin
    scalpel stance --range 4Ki+241664 --output part_A.bin firmware.bin --file-format bin
    scalpel stance --range 282624+241664 --output part_B.bin firmware.hex --file-format hex
    ```

* stitch firmware pieces together such as bootloader and application

    ```bash
    scalpel stitch --binary tmp/test_bytes --offset 0    --binary tmp/test_bytes --offset 2048 --fill-pattern zero --output stitched.bin
    scalpel stitch --binary tmp/test_bytes --offset 2Ki  --binary tmp/test_bytes --offset 0 --fill-pattern one --output stitched.hex --file-format hex
    scalpel stitch --binary tmp/test_bytes --offset 2058 --binary tmp/test_bytes --offset 10 --fill-pattern random --output stitched.bin
    ```

* replace a section with a new file

    ```bash
    scalpel graft --range 1Ki..2Ki --replace tmp/test_cut_out --output cut tmp/test_bytes
    scalpel graft --range 0..2Ki   --replace tmp/test_cut_out --output cut tmp/test_bytes --file-format bin
    scalpel graft --range 1Ki+1Ki  --replace tmp/test_cut_out --output cut tmp/test_bytes
    scalpel graft --range 1Ki+1Ki  --replace tmp/test_cut_out --output cut tmp/test_bytes.hex --file-format hex
    ```

#### Features

* [x] cut off a binary at specific start and end/size
* [ ] Handle endianness of checksums properly
* [x] Replace parts (i.e. cert files or non volatile memory and/or sections)
* [x] Allow hexadecimal input
* [x] Allow multipile input scales (K = 1000, Ki = 1024, M = 1e6, Mi = 1024*1024, ...)
* [ ] Add verifier option for alignment to given sector/page size
* [x] Allow files in IntelHex format for in- and output

#### Common / Hints

* You need the extracted binary as include? Use `xxd -i sliced.bin > sliced_binary.h` to create a header file out of the result.

* Convert RSA keys in .pem format to pkcs8 format via openssl (see `ring` doc [doc-ring] ), `openssl` supports Ed25519 algorithm currently only on `master`

    ```bash
    openssl pkcs8 -toppk8 -nocrypt -outform der -in [key.pem] > [pkcs8_key.pk8]
    ```

* Generate valid Ed25519 Keypair use small tool from `ring` author:

    ```bash
    cargo install kt
    kt generate ed25519 --out=FILE
    ```

[ring]: https://crates.io/crates/ring
[doc-ring]: https://docs.rs/ring/0.13.0-alpha/ring/signature/struct.RSAKeyPair.html
