name: 🔄 Auto-Update Pinned Repos

on:
  schedule:
    - cron: "0 0 * * *"   # runs daily at UTC midnight — same boundary GitHub uses for streaks
  workflow_dispatch:        # allows manual trigger from Actions tab
  push:
    branches: [main]

jobs:
  update-pinned-repos:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: Checkout profile repo
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: pip install requests

      - name: Fetch pinned repos & patch README
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GH_USERNAME: ${{ github.repository_owner }}
        run: |
          python - <<'EOF'
          import os, re, requests, json

          username = os.environ["GH_USERNAME"]
          token    = os.environ["GH_TOKEN"]

          # GitHub GraphQL — fetch up to 6 pinned repos
          query = """
          {
            user(login: "%s") {
              pinnedItems(first: 6, types: REPOSITORY) {
                nodes {
                  ... on Repository {
                    name
                    description
                    url
                  }
                }
              }
            }
          }
          """ % username

          r = requests.post(
              "https://api.github.com/graphql",
              json={"query": query},
              headers={"Authorization": f"Bearer {token}"},
          )
          repos = r.json()["data"]["user"]["pinnedItems"]["nodes"]

          # Build repo card HTML block
          CARD = (
              '[![Readme Card]'
              '(https://github-readme-stats.vercel.app/api/pin/'
              '?username={user}&repo={repo}'
              '&theme=radical&bg_color=0D1117&border_color=BD93F9'
              '&title_color=FF79C6&text_color=F8F8F2&icon_color=8BE9FD)]'
              '({url})'
          )

          cards = "\n".join(
              CARD.format(user=username, repo=rep["name"], url=rep["url"])
              for rep in repos
          )

          block = (
              "<!-- PINNED-REPOS:START -->\n"
              "<div align=\"center\">\n\n"
              + cards + "\n\n"
              "</div>\n"
              "<!-- PINNED-REPOS:END -->"
          )

          with open("README.md", "r") as f:
              content = f.read()

          updated = re.sub(
              r"<!-- PINNED-REPOS:START -->.*?<!-- PINNED-REPOS:END -->",
              block,
              content,
              flags=re.DOTALL,
          )

          with open("README.md", "w") as f:
              f.write(updated)

          print(f"✅ Patched README with {len(repos)} pinned repos.")
          EOF

      - name: Commit & push if changed
        run: |
          git config user.name  "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git diff --quiet && echo "No changes." || (
            git add README.md
            git commit -m "chore: auto-update pinned repos [skip ci]"
            git push
          )
