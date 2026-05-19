name: "Direct Download"

on:
  workflow_dispatch:
    inputs:
      file_url:
        description: "Direct File URL"
        required: true
        type: string

      upload_method:
        description: "Upload Method"
        required: true
        default: "repository"
        type: choice
        options:
          - repository
          - release

      release_tag:
        description: "Release Tag"
        required: false
        default: ""
        type: string

permissions:
  contents: write

env:
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true

jobs:
  direct-download:
    runs-on: ubuntu-latest
    timeout-minutes: 40

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v5
        with:
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Download File
        id: download
        shell: bash
        run: |
          mkdir -p temp_dl

          FILE_URL="${{ github.event.inputs.file_url }}"

          FILENAME=$(basename "$FILE_URL" | cut -d'?' -f1)
          FILENAME=$(echo "$FILENAME" | sed 's/[^a-zA-Z0-9._() -]/_/g')

          echo "Downloading: $FILENAME"

          curl -L --fail --retry 3 \
            -o "temp_dl/$FILENAME" \
            "$FILE_URL"

          echo "filename=$FILENAME" >> "$GITHUB_OUTPUT"
          echo "filepath=temp_dl/$FILENAME" >> "$GITHUB_OUTPUT"

      - name: Get Current Date
        id: get_date
        shell: bash
        run: |
          echo "date=$(date '+%Y-%m-%d %H:%M:%S')" >> "$GITHUB_OUTPUT"

      - name: Push to Repository
        if: github.event.inputs.upload_method == 'repository'
        shell: bash
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"

          mkdir -p downloads

          mv temp_dl/* downloads/ 2>/dev/null || true

          git add downloads/

          git commit -m "Direct Download: ${{ steps.download.outputs.filename }} [skip ci]" || echo "Nothing to commit"

          git push

      - name: Upload to GitHub Releases
        if: github.event.inputs.upload_method == 'release'
        uses: softprops/action-gh-release@v2
        with:
          tag_name: ${{ github.event.inputs.release_tag != '' && github.event.inputs.release_tag || format('direct-{0}', github.run_number) }}
          name: Direct Download Release
          body: |
            Direct download completed.

            File: ${{ steps.download.outputs.filename }}
            Date: ${{ steps.get_date.outputs.date }}
          files: temp_dl/*

        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Cleanup
        if: always()
        shell: bash
        run: |
          rm -rf temp_dl
