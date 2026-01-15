# DDEV Starter

This provides a starting point for DDEV with some opinionated custom commands.

## Script for `.zprofile` or equivalent to "ddevlize" into an existing folder / new project:
```
# Bootstrap a project-local .ddev directory from the starter repo, then set the DDEV project name.
function ddevalize() {
  local repo_url="https://github.com/twfahey1/ddev-starter.git"

  if [ -d ".ddev" ]; then
    echo "./.ddev already exists; not overwriting. Remove it and re-run ddevalize."
    return 1
  fi

  command -v git >/dev/null 2>&1 || { echo "git is required."; return 1; }

  git clone "$repo_url" .ddev || return 1
  rm -rf .ddev/.git

  if [ ! -f ".ddev/config.yaml" ]; then
    echo "Expected .ddev/config.yaml to exist after cloning, but it was not found."
    return 1
  fi

  local raw_name
  echo -n "DDEV project name (e.g., moody-cms): "
  read -r raw_name

  if [ -z "$raw_name" ]; then
    echo "Name is required."
    return 1
  fi

  # DDEV names are typically lowercase and use [a-z0-9-].
  local safe_name
  safe_name=$(echo "$raw_name" \
    | tr '[:upper:] ' '[:lower:]-' \
    | tr -cd 'a-z0-9-' \
    | sed -E 's/-+/-/g; s/^-+//; s/-+$//')

  if [ -z "$safe_name" ]; then
    echo "Provided name did not contain any usable characters after sanitizing."
    return 1
  fi

  if [ "$safe_name" != "$raw_name" ]; then
    echo "Using sanitized name: $safe_name"
  fi

  # macOS sed requires an explicit empty backup suffix for -i.
  sed -i '' "s/^name:.*$/name: ${safe_name}/" .ddev/config.yaml
  echo "Updated .ddev/config.yaml name: ${safe_name}"

  local pantheon_answer
  echo -n "Is this a Pantheon project? [y/N]: "
  read -r pantheon_answer

  if [[ "$pantheon_answer" =~ ^([Yy]|[Yy][Ee][Ss])$ ]]; then
    if command -v ddev >/dev/null 2>&1; then
      echo "Running: ddev sync-db-setup"
      ddev sync-db-setup || { echo "ddev sync-db-setup failed."; return 1; }
    else
      echo "ddev not found; skipping ddev sync-db-setup."
    fi
  fi
}

alias ddevlize=ddevalize

```
