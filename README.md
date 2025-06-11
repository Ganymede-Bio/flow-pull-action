# Flow Pull Action.

GitHub Action to pull the latest version of flow code from Ganymede.

## Description

This action supports pulling all flows or just a single flow from a specified Ganymede environment. It makes an API call to retrieve a signed URL, downloads the flow code as a ZIP file, and extracts it to a specified directory.
You will want to pull all of the flow contents into your repo prior to starting development, then you may want to pull the contents on a regular interval (cron) or on-demand.
## Inputs

| Input | Description | Required |
|-------|-------------|----------|
| `environment` | The Ganymede environment to pull from | Yes |
| `ganymede_subdomain` | The Ganymede subdomain where the environment is located | Yes |
| `ganymede_api_token` | API token for authenticating with Ganymede | Yes |
| `flow_name` | If set, only the specified flow will be pulled. If not set, all flows will be pulled | No |
| `zip_directory` | If set, the flow will be unzipped to the specified directory. If not set, the flow will be unzipped to a directory named after the environment | No |

## Outputs

| Output | Description |
|--------|-------------|
| `zip_dir` | The full path to the directory where the flow was unzipped |

## Example Usage (Basic)
The action in your repo would look like this:
```yaml
jobs:
  pull-flows:
    runs-on: ubuntu-latest
    steps:
      - name: Pull flow from Ganymede
        uses: ganymede/flow-pull-action@v1
        id: pull-flow
        with:
          environment: 'my-environment'
          ganymede_subdomain: 'my-company'
          ganymede_api_token: ${{ secrets.GANYMEDE_API_TOKEN }}
          flow_name: 'my-flow'  # Optional

      - name: Use extracted files
        run: |
          echo "Flow files were extracted to ${{ steps.pull-flow.outputs.zip_dir }}"
```

## Example Usage (Automated PR)

```yaml
env:
    github_username: repo-releaser
    github_email: service-agent@company

jobs:
  pull-flow:
    runs-on: ubuntu-latest
    steps:
      - name: Pull flow from Ganymede
        uses: ganymede/flow-pull-action@v1
        id: pull-flow
        with:
          environment: 'my-environment'
          ganymede_subdomain: 'my-company'
          ganymede_api_token: ${{ secrets.GANYMEDE_API_TOKEN }}

      - name: Commit changes
        id: commit
        run: |
            git config --global user.name "$github_username"
            git config --global user.email "$github_email"
            git add .
            git commit -m "Sync flows in environment ${{ inputs.environment }} from Ganymede"
        continue-on-error: true

      - name: Set branch name with date
        id: branch_name
        run: |
            CURRENT_DATE=$(date '+%Y-%m-%d-%H-%M-%S')
            echo "BRANCH_NAME=sync-${CURRENT_DATE}" >> $GITHUB_ENV

      - name: Create Pull Request
        if: steps.commit.outcome == 'success'
        uses: peter-evans/create-pull-request@v5
        with:
            title: "Sync flows in environment ${{ inputs.environment }} from Ganymede"
            commit-message: "Sync flows in environment ${{ inputs.environment }} from Ganymede"
            body: |
                This PR contains latest files from Ganymede.
                
                Automated PR created by the pull from ganymede workflow.
            branch: ${{ env.BRANCH_NAME }}
            base: main
```

## Example Usage (Daily Sync)

```yaml 

on:
  schedule:
    - cron: "0 9 * * *"

jobs:
  pull-flows:
    runs-on: ubuntu-latest
    steps:
      - name: Pull flow from Ganymede
        uses: ganymede/flow-pull-action@v1
        id: pull-flow
        with:
          environment: 'my-environment'
          ganymede_subdomain: 'my-company'
          ganymede_api_token: ${{ secrets.GANYMEDE_API_TOKEN }}
          flow_name: 'my-flow'  # Optional

      - name: Use extracted files
        run: |
          echo "Flow files were extracted to ${{ steps.pull-flow.outputs.zip_dir }}"
```
