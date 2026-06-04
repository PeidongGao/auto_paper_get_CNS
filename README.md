# Journal Harvester SOP

Browser-based workflows and scripts for extracting bibliographic metadata from
publicly accessible journal web pages and, where applicable, public metadata
services such as Crossref. Current SOPs cover **Cell**, **Nature**, and
**Science**.

## Repository Strategy

- Canonical code repository:
  <https://github.com/PeidongGao/journal-harvester-sop>
- WillGaoLab knowledge-base:
  <https://github.com/WillGaoLab/WillGaoLab>
- WilliamGaoWeb project display:
  <https://github.com/PeidongGao/WilliamGaoWeb>

The canonical repository is maintained through William Gao's personal GitHub
account. WillGaoLab is a separate public-facing brand.

## SOPs

- [Nature metadata export](nature/Nature_SOP_1.0.md)
- [Science metadata export](science/Science_SOP_1.0.md)
- [Cell metadata export](cell/Cell_SOP_1.0.md)

Each SOP is manually initiated in a browser's developer console and exports a
CSV file locally. The scripts are intended for low-volume, user-initiated use,
not large-scale crawling or automated harvesting. Generated CSV datasets are
excluded from this repository.

## Usage Boundaries

This repository does not host publisher PDFs, full-text articles, paywalled
content, copyrighted figures, or bulk redistributed publisher content.

The scripts are intended only for low-volume, manual, user-initiated metadata
export. Users must review and comply with each publisher's current terms,
institutional license agreements, robots/crawling policies, and applicable law
before use.

Read [DISCLAIMER.md](DISCLAIMER.md) before using these SOPs.

## License

Original code and documentation in this repository are available under the
[MIT License](LICENSE). The license does not apply to third-party content,
metadata, names, logos, or trademarks.
