# LQ-Equiv

Project: LQ-Equiv — documentation and tools for LQ equivalence

Summary
This repository provides the main documentation (LQEquiv.pdf), a Windows executable (LQL-Equiv.exe) and the source archive (codesource.zip).

Top-level contents
- `LQEquiv.pdf` — main documentation / paper
- `LQL-Equiv.exe` — compiled Windows executable (use with caution)
- `codesource.zip` — source archive
- `LICENSE` — project license

Quick view / demo
1. View the PDF online: https://github.com/cyrilvoyant/LQ-Equiv/blob/main/LQEquiv.pdf  
2. Recommended: enable GitHub Pages and serve the `docs/` folder to provide a simple demo page that embeds the PDF and links the binaries.

Security note about binaries
- `LQL-Equiv.exe` is a compiled binary. Please verify its origin before running it. Consider providing checksums (SHA256) or building instructions in the future.

Enable GitHub Pages
- Settings → Pages → Source: Branch `main` / folder `/docs`
- Demo will be available at `https://cyrilvoyant.github.io/LQ-Equiv/` after activation.

Contributing
Please open issues for enhancements/bugs. See CONTRIBUTING.md for contribution guidelines.

Planned improvements in this PR
- Add `docs/index.html` demo page to preview LQEquiv.pdf and link the binary/source.
- Add CONTRIBUTING.md with contribution guidelines.
- Add a GitHub Actions workflow to lint Markdown/HTML.
- Add a minimal .gitignore.

License
See LICENSE in the repository.
