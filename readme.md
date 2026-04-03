name: Separate Primary Images

on:
  push:
    paths:
      - '*.zip'

permissions:
  contents: write

jobs:
  separate:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Clean previous output folders
        run: |
          rm -rf all_images primary
          mkdir -p all_images primary

      - name: Unzip renamed images
        run: |
          for zip in *.zip; do
            [ -f "$zip" ] || continue
            echo "Extracting $zip..."
            unzip -o "$zip" -d all_images/
          done

      - name: Copy primary images (-1 files)
        run: |
          find all_images/ -type f \( -iname "*-1.jpg" -o -iname "*-1.jpeg" -o -iname "*-1.png" -o -iname "*-1.webp" \) -exec cp {} primary/ \;
          echo "Primary images found:"
          ls primary/

      - name: Commit results
        run: |
          git config user.name "github-actions"
          git config user.email "actions@github.com"
          git add all_images/ primary/
          git diff --cached --quiet && echo "No changes to commit" && exit 0
          git commit -m "Auto: unzip and separate primaries [skip ci]"
          git push
