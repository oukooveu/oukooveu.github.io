## oukooveu.github.io

Source theme is [so-simple-theme](https://github.com/mmistakes/so-simple-theme) by [mmistakes](https://github.com/mmistakes/).

### How to run this site locally
```
export JEKYLL_VERSION=3.8.5
docker run --rm -v="$PWD:/srv/jekyll" -v="$PWD/.bundle:/usr/local/bundle" -it -p 4000:4000 jekyll/jekyll:$JEKYLL_VERSION jekyll serve
```

Force GitHub site update:
```
git commit --allow-empty -m "force rebuild of site"
```

Fixed github-pages version in Gemfile:
```
gem "github-pages", "~> VERSION", group: :jekyll_plugins
```

### How to check versions supported by GitHub:
```
curl -s https://pages.github.com/versions.json | jq .
```

### Known issues

Tags for collections do not work, there is related [PR](https://github.com/jekyll/jekyll/pull/5857), but it's still not clear how to incorporate this in existing theme.
