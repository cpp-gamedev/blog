## CPP Gamedev Blog

### Running the site locally

The site is built with hugo, which needs to [first be
installed](https://gohugo.io/getting-started/installing/). The extended version
of hugo is required.

The site uses submodules to store the theme, which needs to be inited after
cloneing the repo. This can be done with:

```bash
git clone https://github.com/cpp-gamedev/blog.git
cd blog
git submodule update --init --recursive
```

Then to run the server locally (with drafts)

```bash
hugo server -D
```

Then navigate to http://localhost:1313 to browse the site and see your changes.
The site will automatically reload for any files changes.
