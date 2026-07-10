# Setup Guide — Live Neofetch-Style GitHub Card

This is the same system Andrew Grant's profile uses: a Python script (`today.py`) runs
on a schedule via GitHub Actions, pulls your real stats (repos, stars, commits,
followers, lines of code, account age) from the GitHub GraphQL API, and writes them
directly into `dark_mode.svg` / `light_mode.svg`. Your `README.md` just displays
whichever SVG matches the viewer's theme.

## 1. Create the special repository
GitHub renders a repo's README on your profile page only if the **repo name exactly
matches your username**. So create a new repo named:

```
RohanDawange
```

(public, with a README — you'll overwrite it).

## 2. Upload these files to the repo root
```
README.md
today.py
dark_mode.svg
light_mode.svg
.github/workflows/build.yaml
cache/requirements.txt
```

## 3. Create a Personal Access Token (classic or fine-grained)
Go to **GitHub → Settings → Developer settings → Personal access tokens**.

Fine-grained token permissions needed:
- Account permissions: `Followers` (read), `Starring` (read)
- Repository permissions (All repositories): `Contents` (read), `Metadata` (read),
  `Commit statuses` (read), `Pull requests` (read)

Copy the token — you'll only see it once.

## 4. Add two repository secrets
In your new `RohanDawange` repo: **Settings → Secrets and variables → Actions → New repository secret**

| Secret name | Value |
|---|---|
| `ACCESS_TOKEN` | the personal access token you just created |
| `USER_NAME` | `RohanDawange` |

## 5. Set your real date of birth (for the "Uptime" field)
Open `today.py`, find this line near the bottom:

```python
age_data, age_time = perf_counter(daily_readme, datetime.datetime(2005, 1, 1))
```

Replace `2005, 1, 1` with your actual birth year, month, day.

## 6. Run it
- Push any commit to `main`, **or**
- Go to the **Actions** tab → "README build" → **Run workflow** (manual trigger)

The action will calculate your live stats and commit updated SVGs automatically.
It also re-runs every day at 04:00 UTC via the cron schedule, so your card always
stays current — no manual updates needed.

## 7. Customize the info card
All the personal fields (college, roles, tech stack, contact info) live directly
inside `dark_mode.svg` and `light_mode.svg` as plain `<tspan>` text — edit them like
normal text, just don't touch the `id="..."` attributes on the stat fields, since
`today.py` looks for those exact IDs to inject live numbers.

## Notes
- First run may take 20–60 seconds since it walks your commit history across all repos.
- If a repo has thousands of commits, GitHub's GraphQL API can rate-limit — this is
  normal and the script has built-in retry/caching (`cache/` folder) to speed up
  future runs.
- Want it to also update on every visit? It won't — GitHub Actions on the free tier
  only runs on schedule/push/manual trigger, which is why the daily cron is included.
