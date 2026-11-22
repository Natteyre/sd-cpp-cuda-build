# sd-cli dla Colab T4 — gotowa binarka z GitHub Actions

Ten projekt **nie zawiera kodu** — to tylko przepis na zbudowanie `sd-cli` z
[leejet/stable-diffusion.cpp](https://github.com/leejet/stable-diffusion.cpp) z backendem CUDA
i opublikowanie gotowego zipa. Notatnik Colab pobiera ten zip i nie kompiluje niczego
(25–45 min → ~20 s).

## Dlaczego

W release'ach `stable-diffusion.cpp` **nie ma** prebuilt CUDA dla Linuksa (tylko CPU, Vulkan, ROCm).
Na T4 trzeba kompilować — a Colab free ma 2 vCPU. GitHub Actions ma 4 vCPU i robi to raz,
zamiast za każdym razem.

## Jak użyć (5 minut, jednorazowo)

1. Utwórz **puste, publiczne** repo na GitHubie, np. `sd-cpp-cuda-build`.
2. Wrzuć te dwa pliki (`README.md` + `.github/workflows/build-cuda-t4.yml`) — przez web UI
   „Add file → Upload files", albo z terminala:

   ```bash
   cd "D:/HeremsAgent Projects/sd-cpp-cuda-build"
   git init && git add -A && git commit -m "build sd-cli CUDA sm75"
   git branch -M main
   git remote add origin https://github.com/TWOJ_LOGIN/sd-cpp-cuda-build.git
   git push -u origin main
   ```

3. Actions → **build sd-cli (CUDA)** → **Run workflow** (zostaw domyślne: `master`, `75`).
   Pierwszy build: ~25–40 min. Kolejne (gdy zmienisz `sd_ref`): tyle samo, bez cache.
4. Gdy job przejdzie zielono — zrób Release, żeby mieć **stały** URL:

   ```bash
   git tag v1 && git push --tags
   ```

   Workflow sam doda `sd-cli-cuda12-sm75-linux-x86_64.zip` do release'a.

5. W notatniku Colab ustaw jedną zmienną:

   ```python
   SD_ZIP_URL = "https://github.com/TWOJ_LOGIN/sd-cpp-cuda-build/releases/latest/download/sd-cli-cuda12-sm75-linux-x86_64.zip"
   ```

   Notatnik pobierze zip, rozpakuje i sprawdzi `sd-cli --help`. Jeśli cokolwiek nie zadziała
   (brak release'a, brak biblioteki CUDA, glibc) — **automatycznie wraca do kompilacji ze źródeł**.

## Co jest w zipie

Cały katalog `build/bin`: `sd-cli` + `libstable-diffusion.so` + backendy ggml (`libggml-cuda.so` itd.).
`CMAKE_INSTALL_RPATH='$ORIGIN'` sprawia, że binarka znajduje biblioteki obok siebie — wystarczy
rozpakować i uruchomić. **Nie** pakujemy bibliotek CUDA (`libcublas.so.12`, ~500 MB) — Colab ma je
własne (przez PyTorch i `/usr/local/cuda`).

## Parametry

| Input | Domyślnie | Znaczenie |
|---|---|---|
| `sd_ref` | `master` | tag/gałąź sd.cpp; przypnij tag, jeśli master się zepsuje |
| `cuda_arch` | `75` | T4=75, L4=89, A100=80 |
| `cuda_version` | `12.8.1` | musi być 12.x — CUDA 13 usunęła sm_75 |

## Uwagi

- `ubuntu-22.04`, nie `24.04`: glibc 2.35 działa na obu obrazach Colab, odwrotnie nie.
- Obraz Colab musi mieć sterownik ≥525 (minor version compatibility CUDA 12.x). T4 na Colabie ma 550+.
- Zużycie: ~35 min runnera na build. Publiczne repo = darmowe minuty Actions bez limitu.
