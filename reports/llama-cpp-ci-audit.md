# llama.cpp CI Artifact Audit

**Audit date:** 2026-10-20 (live GitHub data)
**Auditor note:** Prepared for the platform group — quick due-diligence on whether llama.cpp CI publishes downloadable artifacts we could use to avoid compiling ourselves.

---

## 1. Project overview

**llama.cpp** (full name on GitHub: **`ggml-org/llama.cpp`**; the well-known legacy URL `ggerganov/llama.cpp` now redirects there) is an LLM inference engine written in plain C/C++ with no dependencies. Its stated goal is to enable LLM (and VLM) inference with minimal setup and state-of-the-art performance on a wide range of hardware — locally and in the cloud — with first-class Apple silicon support, AVX/AVX2/AVX512/AMX on x86, broad GPU backends (CUDA, HIP, Metal, Vulkan, SYCL, MUSA, CANN, and more), and 1.5–8-bit integer quantization for faster inference and reduced memory use. It ships a CLI, an OpenAI-compatible server (`llama-server`), and is built on the [ggml](https://github.com/ggml-org/ggml) tensor library.

- **License:** MIT (as reported by the GitHub repository metadata, and the MIT badge in the README)
- **Default branch:** `master`

## 2. CI inspection

### Workflow inspected

**"CI (cpu)"** — workflow file `.github/workflows/build-cpu.yml` (workflow id 284408720).

This is the repository's core/primary build-and-test workflow: it triggers on pushes to `master` and on pull requests touching source/build files, and it builds llama.cpp with CMake and runs the test suites (`ctest -L 'main|python'`, plus a llama2c-conversion smoke test) across an **ubuntu** matrix (x64 + arm64) and a **windows** matrix (x64-cpu-static, x64-openblas, arm64). It also invokes the reusable `build-cmake-pkg.yml` ("Build relocatable cmake package") job to verify that a relocatable CMake package can be built and consumed by a downstream example project.

### Run inspected

- **Workflow:** CI (cpu)
- **Run number:** #11337
- **Run id:** 39380213267
- **Run URL:** https://github.com/ggml-org/llama.cpp/actions/runs/39380213267
- **Event / branch:** `pull_request` (head branch `hexagon-toolchain-bump`)
- **Commit:** `89e7cee8335043434f6443a5fa0ce2c75edddd52` — "hexagon: update to snapdragon-toolchain v0.8 and enable hexagon v85"
- **Outcome:** **completed / success**

### Artifacts

**No artifacts were published by this run.**

The run's artifact list is empty. Because the newest run had no artifacts, the audit walked backwards through earlier runs, and **all** of the five most recent runs of the workflow were checked:

| Run number | Run id | Trigger / branch | Conclusion | Artifacts |
| --- | --- | --- | --- | --- |
| #11337 | 39380213267 | PR `hexagon-toolchain-bump` | success | 0 |
| #11336 | 39379258566 | PR `iq3_xxs-avx512vl-dot` | action_required | 0 |
| #11335 | 39377291250 | PR `gemma4v-vision-gelu` | action_required | 0 |
| #11334 | 39374997964 | PR `cpu-mul-mat-f16-src1` | action_required | 0 |
| #11333 | 39371435186 | push to `master` | success | 0 |

(For reference, run #11337 as originally seen in-flight — id 39380213267 — completed with conclusion `success` shortly after the audit began; the run in the table above is the same one.)

**Root cause (confirmed at workflow-definition level):** `build-cpu.yml` contains **no `upload-artifact` steps at all** — the "CI (cpu)" workflow compiles and tests the code purely for verification and discards the binaries. A code search across `.github/workflows/` confirms only a small set of release/packaging workflows (`release.yml`, `build-apple.yml`, `build-android.yml`, `ui-build.yml`) use `actions/upload-artifact`; the main build/test workflows do not.

- **Artifact name:** none
- **Artifact size:** none
- **Artifact download URL:** none

## 3. Where llama.cpp binaries actually come from

Although the CI build/test workflow publishes no downloadable artifacts, prebuilt binaries **are** available from llama.cpp, just not from CI:

1. **Releases page** — https://github.com/ggml-org/llama.cpp/releases. The separate **"Release"** workflow (`.github/workflows/release.yml`) runs on pushes to `master`, packs per-platform `llama-<tag>-bin-<os>-<arch>.tar.gz` tarballs (macOS arm64/x64, Ubuntu x64/arm64/s390x, Windows, and more) and publishes them to GitHub Releases (with `b*` nightly tags). This is the official, supported channel for prebuilt binaries.
2. **Installers** — `curl -LsSf https://llama.app/install.sh | sh` (and the PowerShell equivalent), which fetch the release binaries.
3. **Docker images** — published by the "Publish Docker image" workflow (`docker.yml`).

## 4. Bottom line for the platform group

You **cannot** pull prebuilt binaries from llama.cpp's CI runs — the main build/test workflow ("CI (cpu)") is verification-only and uploads nothing. If we want llama.cpp binaries without compiling, the supported route is the GitHub **Releases** page (or the `llama.app` installer), which publishes fresh per-platform tarballs on every push to `master`.

---

*Data collected live from the GitHub API on 2026-10-20.*
