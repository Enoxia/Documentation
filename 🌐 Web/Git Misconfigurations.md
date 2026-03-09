# .git Found

If we found a `.git` folder from a URL, we can use [git-dumper](https://github.com/arthaud/git-dumper) to dump it 
```bash
git-dumper http://website.com/.git ~/website
```

We can also use a [rewrite](https://github.com/holly-hacker/git-dumper) of the tool, which is 10x faster
```bash
python3 git-dumper.py https://victim/.git/ out && cd out && git checkout -- .
```

Once downloaded, we can use [GitKraken](https://www.gitkraken.com/) to inspect its content.

The tool [GitDump](https://github.com/Ebryx/GitDump) brute-forces object names from `.git/index, packed-refs, etc.` to recover repos even when directory traversal is disabled :
```bash
python3 git-dump.py https://victim/.git/ dump && cd dump && git checkout -- .
```

=> If a .git directory is found in a web application you can download all the content using `wget -r http://web.com/.git`. Then, you can see the changes made by using `git diff`.

- The tools: [Git-Money](https://github.com/dnoiz1/git-money), [DVCS-Pillage](https://github.com/evilpacket/DVCS-Pillage) and [GitTools](https://github.com/internetwache/GitTools) can be used to retrieve the content of a git directory.
- The tool [Git-Vuln-Finder](https://github.com/cve-search/git-vuln-finder) can be used to search for CVEs and security vulnerability messages inside commits messages.
- The tool [GitRob](https://github.com/michenriksen/gitrob) search for sensitive data in the repositories of an organisations and its employees.
- [Repo security scanner](https://github.com/UKHomeOffice/repo-security-scanner) is a command line-based tool that was written with a single goal: to help you discover GitHub secrets that developers accidentally made by pushing sensitive data. And like the others, it will help you find passwords, private keys, usernames, tokens and more.
- [Here](https://securitytrails.com/blog/github-dorks) we can find an study about github dorks

## Git post-dump
```bash
cd dumpdir

# Reconstruct working tree
git checkout -- .

# Show branch/commit map
git log --graph --oneline --decorate --all

# list suspicious config/remotes/hooks
git config -l
ls .git/hooks
```

## Credentials Hunting
- [GitLeaks](https://github.com/gitleaks/gitleaks) is a tool for detecting secrets like passwords, API keys, and tokens in git repos, files, and whatever else you wanna throw at it via stdin.
- [TruffleHog](https://github.com/trufflesecurity/trufflehog) is designed to detect and verify secrets in codebases with high accuracy.

## Server-Side integration RCE via hooksPath override
Modern web apps that integrate Git repos sometimes `rewrite .git/config using user-controlled identifiers`. If those identifiers are concatenated into hooksPath, you can redirect Git hooks to an attacker-controlled directory and execute arbitrary code when the server runs native Git (e.g., git commit).
Key steps:

- `Path traversal in hooksPath`: if a repo name/dependency name is copied into hooksPath, inject ../../.. to escape the intended hooks directory and point to a writable location. This is effectively a [path traversal](https://book.hacktricks.wiki/en/pentesting-web/file-inclusion/README.html) in Git config.
- `Force the target directory to exist`: when the application performs server-side clones, abuse clone destination controls (e.g., a ref/branch/path parameter) to make it clone into ../../git_hooks or a similar traversal path so intermediate folders are created for you.
- `Ship executable hooks`: set the executable bit inside Git metadata so every clone writes the hook with mode 100755:
git update-index --chmod=+x pre-commit
Add your payload (reverse shell, file dropper, etc.) to pre-commit/post-commit in that repo.
- `Find a native Git code path`: libraries like JGit ignore hooks. Hunt for deployment flows/flags that fall back to system Git (e.g., forcing deploy-with-attached-repo parameters) so hooks will actually run.
- `Race the config rewrite`: if the app sanitizes .git/config right before running Git, spam the endpoint that writes your malicious hooksPath while triggering the Git action to win a [race condition](https://book.hacktricks.wiki/en/pentesting-web/race-condition.html) and get your hook executed.
