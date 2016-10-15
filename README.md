zddhub's blog
-------------

This is my blog, I am writing some things just for fun.


### Blogs and Jekyll pages separation

I want a clean workspace to write, only includes plain text (such as: markdown file) and some images used in the article.

Jekyll is a good choice but it mixed plain text and websites, so ...

My site is separated into two parts: 

- [Blogs](https://github.com/zddhub/zddhub.github.io/tree/blogs) - only include markdown file and some images
- [Jekyll pages](https://github.com/zddhub/zddhub.github.io/tree/master) - Jekyll pages, html, layout, css, ...


### Workflow

1. Clone repo and work on `blogs` branch (once at first time)

    git clone https://github.com/zddhub/zddhub.github.io.git
    git checkout -t origin/blogs

2. Write post and push to blogs branch.

3. Run `./build_pages` to trigger Jekyll pages on Github side.


### How to work

1. Prepare Blogs data and push on blogs branch.
2. Add submodule to Jekyll site.

    git submodule add -b blogs https://github.com/zddhub/zddhub.github.io .blogs

3. Symlink a submodule files to Jekyll site.

    ln -s .blogs/posts _posts
    ln -s .blogs/asserts asserts
    ln -s .blogs/about.md about.md

### Thanks
Powered by [jekyll](http://jekyllrb.com/) and [lanyon](http://lanyon.getpoole.com/).
