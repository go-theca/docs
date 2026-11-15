# docs

Documentation of [go-theca](https://github.com/go-theca), built with Hugo
and the [tannevaled/hextra](https://github.com/tannevaled/hextra) fork (a submodule
pinned at the release tag `v0.24.1`).

Published at <https://go-theca.github.io/docs/>: an overview and one page per
repository of the organisation, each written from the repository's README (the tag
is named on the page; a repository with no tag says so).

```sh
git clone --recurse-submodules https://github.com/go-theca/docs
hugo server
```

Build as CI does: `hugo --gc --minify --panicOnWarning --printPathWarnings`.
