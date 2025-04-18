name: Update Index Versions

on:
  release:
    types: [published]

jobs:
  update-index:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Get latest release version
        id: get_version
        run: echo "VERSION=$(gh release view --json tagName --jq .tagName)" >> $GITHUB_ENV
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: List available versions
        id: list_versions
        run: |
          FR_VERSIONS=$(ls version/fr | grep -E '^[0-9]+\.[0-9]+\.md$' | sed 's/\.md//g' | sort -V | sed 's/^/[v&](version\/fr\/&.md)/' | tr '\n' ', ' | sed 's/, $//')
          EN_VERSIONS=$(ls version/en | grep -E '^[0-9]+\.[0-9]+\.md$' | sed 's/\.md//g' | sort -V | sed 's/^/[v&](version\/en\/&.md)/' | tr '\n' ', ' | sed 's/, $//')
          echo "FR_VERSIONS=$FR_VERSIONS" >> $GITHUB_ENV
          echo "EN_VERSIONS=$EN_VERSIONS" >> $GITHUB_ENV

      - name: Update index.md
        run: |
          cat > index.md << EOF
# LibreShare Non-Commercial License (LSNC)

Welcome to the official home of the **LibreShare Non-Commercial License ([LSNC v${{ env.VERSION }}](version/fr/${{ env.VERSION }}.md))** at [https://lameur.github.io/LSNC-License](https://lameur.github.io/LSNC-License). This open-source license, designed for Lameur’s projects ([GitHub](https://github.com/Lameur)) and available for use by anyone, promotes the free sharing of source code while prohibiting commercial use and protecting compiled versions against reverse engineering.

## About the LSNC

The LSNC is a versatile open-source license that balances the principles of free software with the protection of proprietary elements, such as non-open-source compilers. It is ideal for developers who want to share their source code under a copyleft model while restricting commercial exploitation.

### Key Features

- **Non-Commercial Use**: Authorized, including for educational purposes (e.g., schools, universities, training).
- **Copyleft**: Derivative works must be shared under the same license with source code.
- **Protection of Compiled Versions**: Reverse engineering is prohibited to safeguard proprietary elements.
- **Flexibility**: Users can request permissions for specific uses (e.g., commercial).

## Available Versions

The current version of the LSNC is **[v${{ env.VERSION }}](version/fr/${{ env.VERSION }}.md)**.

- **French**: ${{ env.FR_VERSIONS }}
- **English**: ${{ env.EN_VERSIONS }}
- **Latest Version**:
  - French: [version/fr/latest.md](version/fr/latest.md)
  - English: [version/en/latest.md](version/en/latest.md)

*Note*: This section is automatically updated when new versions are released.

## Documentation

Learn how to use the LSNC and understand its terms:

- [Usage Guide](docs/usage.md): How to apply the LSNC to your project.
- [FAQ](docs/faq.md): Answers to common questions about the license.

## Contact

For questions, permission requests (e.g., for commercial use), or inquiries about compatibility with other licenses, contact:

- **Email**: [lameur.tech@gmail.com](mailto:lameur.tech@gmail.com)
- **GitHub**: [github.com/Lameur](https://github.com/Lameur)

*Note*: Contact information will be updated once the official website is available.

## Contribute

We welcome suggestions to improve the LSNC! Open an [issue](https://github.com/Lameur/LSNC-License/issues) or submit a [pull request](https://github.com/Lameur/LSNC-License/pulls) on [GitHub](https://github.com/Lameur).

---

© 2025 Lameur
EOF

      - name: Commit changes
        run: |
          git config user.name "GitHub Action"
          git config user.email "action@github.com"
          git add index.md
          git commit -m "Update index.md with available versions for release v${{ env.VERSION }}"
          git push