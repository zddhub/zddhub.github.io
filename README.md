zddhub's blog
-------------

I want a clean workspace to write, only includes plain text (such as: markdown file) and some images used in the article.

Jekyll is a good choice but it mixed plain text and websites, so ...


### Blogs and Jekyll pages separation

My site is separated into two parts:

- [Blogs](https://github.com/zddhub/zddhub.github.io/tree/blogs) - only include markdown file and some images
- [Jekyll pages](https://github.com/zddhub/zddhub.github.io/tree/master) - Jekyll pages, html, layout, css, ...


### Workflow

* Clone repo and work on `blogs` branch (once at first time)

```sh
    git clone https://github.com/zddhub/zddhub.github.io.git master
    git checkout -t origin/blogs
```

* Write post and push to blogs branch.

* Run `./build_pages` to trigger Jekyll pages on Github side.


### How to work

* Prepare Blogs data and push on blogs branch.
* Add submodule to Jekyll site.

```sh
    git submodule add -b blogs https://github.com/zddhub/zddhub.github.io .blogs
```

* Symlink a submodule files to Jekyll site.

```sh
    ln -s .blogs/posts _posts
    ln -s .blogs/asserts asserts
    ln -s .blogs/about.md about.md
```


### Troubleshooting

If you want debug in local (master branch) with `jekyll server --watch`, you will find some INFO like this:

```sh
    ** ERROR: directory is already being watched! **
```

Don't worry, this is just a Warning.


### Thanks
Powered by [jekyll](http://jekyllrb.com/) and [lanyon](http://lanyon.getpoole.com/).
