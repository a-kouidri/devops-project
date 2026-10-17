# TP1 answers

Team members: Aicha Kouidri, <name 2>, <name 3>, <name 4>

## Exercise 1

(a) The two files that start with a dot are `.env.example` and `.gitignore` (the folder `.git` is created by `git init`).

(b) With `cp -r .../ctf-scoreboard/* .` the shell expands `*` and skips names that start with a dot. So `.gitignore` and `.env.example` are not copied. Without `.gitignore`, files like `.env` or `data/scoreboard.db` could be committed by mistake.

(c) `.gitignore` only affects untracked files. If `.env` is already in a commit, Git already tracks it, so the new rule changes nothing, and the file stays in the old commits. It needs `git rm --cached .env`, and even then the history still contains it.

## Exercise 2

(a) Both branches changed the same line (`points: 500` of `the-long-game`) from the same parent, one to 450 and one to 600. Git cannot know which value is correct, so it asks me to choose.

(b) Two changes to two different challenges are on different lines (different hunks), so Git merges them automatically.

(c) The check can run in a local `pre-commit` hook, and in CI (for example a GitHub Actions job on every push and pull request) so that it cannot be skipped.

## Exercise 3

(a) `git bisect run` needed 5 tests for the 31 commits.

(b) ceil(log2 31) = 5, the same number: each step cuts the search range in half.

(c) If `check.sh` is inside the repository, it does not exist in the old commits. The script then fails (exit 127, file not found), bisect counts this as a bad commit, and the result is wrong.

(d) If the 31 commits are squashed into one, bisect can only return that single commit as the first bad one. It does not say which change inside it is the bug.

## Exercise 4

(a) `gitleaks dir .` gives exit=0 (no leak in the working tree). `gitleaks git -v .` gives exit=1 (one leak found in the history).

(b) The rule is `aws-access-token`.

(c) Each commit stores the hash of its parent, so the history is a chain. The second commit does not delete the first one: it adds a new commit on top. The first commit, with the key, is still in the chain and can be read with `git log -p` or `git checkout`.

(d) First, revoke and rotate the key at the provider (for example AWS) and create a new one. Second, clean the history (for example with `git filter-repo`) and force push. The order matters because a pushed key must be considered already stolen: only the revocation makes it useless, and cleaning the history cannot remove copies already taken.

## Exercise 5

(a) Branch protection is also a security control: every change to `main` must go through a pull request with an independent review, so no single account can push code alone. It also blocks force pushes and deletion, so nobody can rewrite or erase the history to hide a change. A stolen account or a careless contributor cannot silently modify the main branch.

(b) It makes harder a malicious or unreviewed commit pushed directly to `main` (a supply chain attack, for example through a compromised account), and the rewriting of history with a force push to hide it.

(c) On a free GitHub account, branch protection rules are available only for public repositories. Private repositories need a paid plan for this feature.
