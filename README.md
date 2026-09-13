# Boiler plate: your own OS based on the CuOS system

A starting point for the
[Own OS based on the CuOS system](https://github.com/cuos-dev/cuos/blob/HEAD/docs/development-guide.md#own-os-based-on-the-cuos-system)
level — you need another target board, or your own kernel drivers, and want to
keep everything else CuOS brings: boot loader, init, API, console menu, updater.

Before you start: nearly everything can be packed into docker containers, host
services and host network configuration included. Only reach for this level when
something genuinely has to live in the OS image.

| File | What it is |
|---|---|
| `system/Dockerfile` | Your system image, built `FROM` the published CuOS system image. |
| `updater/Dockerfile` | Your updater image, built `FROM` the published CuOS updater. |

## Use it

1. Fork or copy this repository.
2. Replace `vx.x.x` in both Dockerfiles with the CuOS version you are building
   against — the one `cuos-release/release.json` pins.
3. Add your packages, kernel and drivers. Anything you drop into
   `/usr/local/cuos/` overrides the CuOS script of that name, which is how a new
   board's `install-kernel-*.sh` gets in.
4. Publish both images, then name them in your `system.json`:

```json
{
  "hostname": "my-system",
  "os_image": "ghcr.io/my-org/my-system",
  "os_image_version": "1.0.0",
  "os_image_digest": "sha256:...",
  "updater_image": "ghcr.io/my-org/my-updater",
  "updater_image_version": "1.0.0",
  "updater_image_digest": "sha256:..."
}
```

No `#include` of `cuos-release/release.json` here: it pins the CuOS images, and
you are publishing your own. Pin yours the same way instead — the `_digest` keys
are what make a build reproducible, and a mismatch is fatal rather than
silently accepted. `docker inspect --format='{{index .RepoDigests 0}}' IMAGE`
prints one.

You still need a **CuOS Init App** in `init_image` — your own, or CuOS IaC. That
is the level above; see the
[Development Guide](https://github.com/cuos-dev/cuos/blob/HEAD/docs/development-guide.md).

For a board rather than a plain x86 machine, use `<platform>_image` instead of
`os_image` and build with `--platform`. A platform the tooling does not know also
needs its disk layout stated — `--layout mbr` or `--layout gpt` — because the
wrong one produces an image that builds cleanly and never boots.

5. Build the system with
   [cuos-release](https://github.com/cuos-dev/cuos-release#readme).

## Next

- [Platform support](https://github.com/cuos-dev/cuos/blob/HEAD/docs/common/platform-support.md)
  — the existing targets and boot chains, and how to add one
- [What a CuOS image contains](https://github.com/cuos-dev/cuos/blob/HEAD/docs/common/building-images.md)
  — the partition layouts and what varies per platform
- [Development Guide](https://github.com/cuos-dev/cuos/blob/HEAD/docs/development-guide.md)

---

## License

**MIT-0** — see [LICENSE.txt](LICENSE.txt). MIT without the attribution clause:
copy this, change it, ship it, and you owe nothing — no notice to carry, no
credit to give.

That is deliberate. This repository exists to be taken. CuOS itself is
Apache-2.0; only the boiler plates and the example are MIT-0.
