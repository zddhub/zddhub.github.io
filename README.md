zddhub's blog
-------------

I want to write in a clean workspace, that only includes plain text (such as: markdown file) and assets referenced in articles.

Gem-based Jekyll theme is a good choice but Github doesn't support customized theme now ([Supported theme list](https://pages.github.com/themes/)).

so ...


### Blogs and Jekyll pages separation solution

My site is separated into three parts:

- [Blogs](https://github.com/zddhub/zddhub.github.io/tree/blogs) - Only include markdown files and assets (also support gem-based theme)
- [Gravid](https://github.com/zddhub/gravid) - A simple and beautiful jekyll theme.
- [Jekyll pages](https://github.com/zddhub/zddhub.github.io/tree/master) - Above two parts are included by `git submodule`, and this repo just link important files for Jekyll pages.


### How to do

* Add submodules to Jekyll site.

```sh
    git submodule add -b blogs https://github.com/zddhub/zddhub.github.io .blogs
    git submodule add -b master https://github.com/zddhub/gravid .gravid
```

* Symlink important files to Jekyll site.

```sh
    ln -s .blogs/posts _posts
    ln -s .blogs/assets assets
    ln -s .blogs/about.md about.md
    ln -s .blogs/archive.md archive.md
    ln -s .blogs/index.md index.md
    ln -s .blogs/assets assets
    ln -s .blogs/_config.yml _config.yml

    mkdir -p _includes # the symlinked files under `includes` cannot be used.
    ln -s .gravid/_layouts _layouts
    ln -s .gravid/_sass _sass
    ln -s .gravid/404.html 404.html
```


### Usage

* Clone repo with submodules (once at first time)

```sh
    git clone --recursive https://github.com/zddhub/zddhub.github.io.git
```

* After you updated and pushed [Blogs](https://github.com/zddhub/zddhub.github.io/tree/blogs) or [Gravid](https://github.com/zddhub/gravid), run `./publish_blog` to trigger Jekyll pages on Github side.


### Gem-based blog

If Github page supports customized theme later, or in your local environment, You can use gem-based style.

```sh
git clone https://github.com/zddhub/zddhub.github.io.git -b blogs

```

And add this line to your Jekyll site's `_config.yml`:

```yaml
theme: gravid
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install gravid

Run `bundle exec jekyll serve` and open your browser at `http://localhost:4000`.


### Troubleshooting

If you want debug in local (master branch) with `jekyll server --watch`, you will find some INFO like this:

```sh
    ** ERROR: directory is already being watched! **
```

Don't worry, this is just a Warning.
