# Day 02 – Image sizes

Built from `aiops-training/day-02-docker-basics` on 2026-11-10 (Docker 29.9.3).

| Image | Size | Layer count | Has a shell? |
|---|---|---|---|
| `hello-py:1.0` | 177 MB (177,347,333 bytes) | 14 | yes |
| `hello-java:1.0` | 457 MB (456,691,982 bytes) | 19 | yes |
| `hello-java:jlink` | 180 MB (179,843,343 bytes) | 8 | yes |
| `hello-go:1.0` | 2.09 MB (2,083,112 bytes) | 2 | no |

- **Size:** `docker image inspect IMAGE --format '{{.Size}}'`.
- **Layer count:** `docker history IMAGE | wc -l` minus the header line. This counts every history row, including 0-byte metadata steps such as `CMD`, `ENV` and `WORKDIR`.
- **Has a shell?:** `docker run --rm --entrypoint /bin/sh IMAGE -c 'echo ok'` succeeds or fails.
- All four images ran and printed `Hello, world!` (Python 3.12.16, Java 21.0.13 ×2, Go 1.23.12).

## Which is smallest, and why

`hello-go:1.0` is the smallest by far: 2 MB, about 85× smaller than the jlink Java image and over 200× smaller than the JRE image. Go compiles to a single static binary (`CGO_ENABLED=0`, symbols stripped with `-s -w`), so the final stage is `FROM scratch`: an empty image holding only that binary. The Python and Java images have to ship an interpreter or JVM plus a Debian/Ubuntu base. jlink cuts Java from 457 MB to 180 MB by building a runtime with only `java.base`, but it still sits on a full Ubuntu 24.04 base.

What you give up with `scratch`: there's no shell, package manager or debugging tools, so `docker exec … sh` doesn't work. There are also no CA certificates, timezone data or `/etc/passwd` unless you copy them in. To debug, you have to rely on logs or attach a separate debug container, for example `kubectl debug`.

For a production Go service I'd choose the `scratch`-style image, or `gcr.io/distroless/static` if the service needs CA certificates and a non-root user out of the box. It's small, pulls fast, and has almost no attack surface.
