name: Update contributions (SVG)

on:
  schedule:
    - cron: "17 2 * * *"   # daily 02:17 UTC
  workflow_dispatch:

permissions:
  contents: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Generate SVG + JSON
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.GH_STATS_TOKEN }}
          script: |
            const fs = require('fs');

            // last 12 months window (matches GitHub calendar)
            const to = new Date();
            const from = new Date(to);
            from.setFullYear(from.getFullYear() - 1);

            const q = `
              query($from: DateTime!, $to: DateTime!) {
                viewer {
                  login
                  contributionsCollection(from: $from, to: $to, includePrivateContributions: true) {
                    contributionCalendar { totalContributions }
                    restrictedContributionsCount
                    totalCommitContributions
                    totalIssueContributions
                    totalPullRequestContributions
                    totalPullRequestReviewContributions
                  }
                }
              }`;

            const res = await github.graphql(q, { from: from.toISOString(), to: to.toISOString() });
            const v = res.viewer.contributionsCollection;

            const fmt = n => n.toLocaleString('en-US');
            const total = v.contributionCalendar.totalContributions;
            const priv  = v.restrictedContributionsCount;

            const line = `${fmt(total)} contributions · commits ${fmt(v.totalCommitContributions)} · PRs ${fmt(v.totalPullRequestContributions)} · issues ${fmt(v.totalIssueContributions)} · reviews ${fmt(v.totalPullRequestReviewContributions)} · private ${fmt(priv)}`;
            const color = '#2E3D48';

            const svg = `
              <svg xmlns="http://www.w3.org/2000/svg" width="720" height="44" viewBox="0 0 720 44">
                <style>
                  .t{font:12px/1.4 -apple-system,BlinkMacSystemFont,Segoe UI,Helvetica,Arial,sans-serif; fill:${color}}
                  .b{fill:none;stroke:#000;stroke-width:1}
                </style>
                <rect width="100%" height="100%" class="b"/>
                <text x="10" y="28" class="t">${line}</text>
              </svg>
            `;

            fs.mkdirSync('assets', { recursive: true });
            fs.writeFileSync('assets/gh_contrib.svg', svg.trim());
            fs.writeFileSync('assets/gh_contrib.json', JSON.stringify({
              updated: new Date().toISOString(),
              last12m_total: total,
              last12m_private: priv,
              commits: v.totalCommitContributions,
              prs: v.totalPullRequestContributions,
              issues: v.totalIssueContributions,
              reviews: v.totalPullRequestReviewContributions
            }, null, 2));

      - name: Commit updated files
        uses: stefanzweifel/git-auto-commit-action@v5
        with:
          commit_message: "chore: update contributions stats"
          file_pattern: assets/gh_contrib.*
