## github actions

- PR의 label 이 data-request 일 때
- PR에서 변경되는 파일은 "data-images", "data-files" 폴더 뿐인지 검증한다.

```
name: user-pr-validator
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_run:
    workflows: [ "Receive PR" ]
    types:
      - completed
jobs:
  build:
    runs-on: ubuntu-latest
    if: contains(github.event.pull_request.labels.*.name, 'data-request')
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0
      - name: Get changed files
        id: changed-files
        uses: tj-actions/changed-files@v14.6
      - name: List all changed files
        run: |
          for file in ${{ steps.changed-files.outputs.all_changed_files }}; do
            echo "$file was changed"
          done
      - name: Check for changes in specific files
        run: |
          ALLOWED_FOLDERS=("data-images" "data-files")
          for file in ${{ steps.changed-files.outputs.all_changed_files }}; do
            folder_path=$(dirname "$file")
            echo "CHANGED_FOLDERS : $folder_path"
            if [[ ! " ${ALLOWED_FOLDERS[@]} " =~ " $folder_path " ]]; then
              echo "Error: Changes to $file are not allowed."
              exit 1
            fi
          done

```
