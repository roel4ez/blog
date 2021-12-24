---
author: Roel Fauconnier
author_gh_user: roel4ez
read_time: 5min
publish_date: 22-dec-2021
tags:
    - blog
    - CI
    - mike
title: Awesome documentation with mkdocs and GitHub Pages
---

Documentation. Developers either love it or hate it. **Love it** when it is
available when they are trying to use the latest and greatest framework (guilty!),
or **hate it** when they have to write it themselves.

Luckily there are some really nice tools out there to manage the documentation
for your projects. [DocFX](https://dotnet.github.io/docfx/) is one of them, this
is the engine that is running the [docs.microsoft.com](https://docs.microsoft.com/en-us/)
sites.

However, this article is about [mkdocs](https://www.mkdocs.org/). This is a neat
little tool that can help you create a static site for your documentation. And it
integrates nicely with GitHub Pages. Let's have a look at how to set up
documentation with mkdocs and mkdocs-material. Later we'll have a look at using
[mike](https://github.com/jimporter/mike) to manage versions.

## Requirements

First of all you'll need Python on your machine, because this is what powers the
whole thing. So go ahead and [install that](https://realpython.com/installing-python/)
and also install [pip](https://pip.pypa.io/en/stable/installation/) while you are
at it. Once you have this, you can install the prerequisites:

```bash
- pip install mkdocs
- pip install mkdocs-material
- pip install mike
```

There will be a couple of other extensions and plugins that you can use, and it
probably makes sense to manage that in a [requirements.txt](https://pip.pypa.io/en/latest/reference/requirements-file-format/) file.

### Branching

Let's say you have a `main` branch where all your code lives. You might be
inclined to create a `/docs` folder there and start dumping your `md` files there.
However, I recommend to keep your docs in a separate repository, so it can be
maintained and managed individually (e.g. permissions). You do not need to setup
a new GitHub repository for that, you can instead create a [detached branch](https://stackoverflow.com/questions/19980631/what-is-git-checkout-orphan-used-for)
or [orphan branch](https://git-scm.com/docs/git-checkout#Documentation/git-checkout.txt---orphanltnewbranchgt)
, called `docs/main` for example:

```bash
git checkout docs/main --orphan
```

If you want to have `dev` branch and `docs` branch side by side, try out 
git worktree

```bash title="git worktree"
git checkout docs/main

# from the working folder:
git worktree add c:/path-to-sources/repository.docs docs/main
```

## Get started

For theming the documentation, I'm using [mkdocs-material](https://squidfunk.github.io/mkdocs-material/).
They have a pretty good guide on how to [get started](https://squidfunk.github.io/mkdocs-material/getting-started/),
so I won't repeat all of that here. Instead, we'll skip to building and deploying

### Working locally

For local development, use the docker-container provided by `mkdocs-material`.
Run `docker run --rm -it -p 8000:8000 -v ${PWD}:/docs squidfunk/mkdocs-material`
in the root directory of the docs to get a copy running on `http://localhost:8000`.
You can change the port in the `docker run` command if needed.

<!-- markdownlint-disable MD046 -->
!!! warning "Required extensions"
    If you are using extensions which are not supported by the mkdocs-material
    container out-of-the-box, you have two ways to deal with this:  

    1. use the [manual approach](#alternate-approach)
    2. Create a [custom docker image](https://squidfunk.github.io/mkdocs-material/getting-started/#with-docker) with the plugin installed:  

    ```dockerfile title="Dockerfile"
    FROM squidfunk/mkdocs-material
    RUN pip install mdx_truly_sane_lists

    # or copy your requirements.txt
    COPY requirements.txt .
    RUN pip install -r requirements.txt
    ```

    ```bash title="Build and run container"
    # in the directory where your dockerfile is
    docker build . -t mkdocs-material-with-extensions
    docker run --rm -it -p 8000:8000 -v ${PWD}:/docs mkdocs-material-with-extensions
    ```
<!-- markdownlint-enable MD046 -->

#### Alternate approach

Install Python and pip, and then the required packages:

```bash
pip install -r requirements.txt
mkdocs serve
```

### Configuration

The file `mkdocs.yml` provides the main configuration for the website, such as 
color and themes, plugins and extension. The `Table of Contents` is also defined
in the config file, under the section `nav`. This is where you can build the
structure of your documentation page.

## Automate it!

GitHub has two cool features we can leverage for our documentation:

1. GitHub Actions: this let's us automate all the things
2. GitHub Pages: a place to host a static site, hosted at `http://<owner>.github.io/<repo>`

So let's do this: once you have something working locally, you can create a GitHub
Action to do all the work for your. Luckily, `mkdocs` comes with a [feature](https://www.mkdocs.org/user-guide/deploying-your-docs/#github-pages)
to help us out: `mkdocs gh-deploy`. This command will push to a branch called
`gh-pages`, which is the default branch GitHub uses to host the files for the
GitHub Pages (it's also an orphaned branch by the way). This means the action
would look something like this:

```yml
name: deploy 
on:
  push:
    branches: 
      - docs/main # deploy on pushes to your documentation branch
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0
      
      - uses: actions/setup-python@v2
        with:
          python-version: 3.x

      - run: pip install -r requirements.txt

      - run: mkdocs gh-deploy --force
```

This is also the exact [same action](https://github.com/roel4ez/blog/blob/main/.github/workflows/deploy.yml) that powers the deployment of this blog!

### Versioning

If you want to take things a bit further and provide versioning in your documentation,
checkout the tool [mike](https://github.com/jimporter/mike). This tool allows you
to deploy multiple versions of your documentation. It sits nicely on top of `mkdocs`
and integrates with everything that was discussed here. It also integrates nicely
with GitHub Pages, which is very cool.

## Conclusion

Besides what is mentioned here, there are of course also other tools that can
achieve the same things, for example [Jekyll](https://jekyllrb.com/).
Choose what works for you.

To see all of this in action, checkout the repository where I have set all of
this up: [iotedge-lorawan-starterkit](https://github.com/Azure/iotedge-lorawan-starterkit/tree/docs/main).
That is a very cool thing in itself, but a topic for another time! Feel free to
take a look at the [Actions](https://github.com/Azure/iotedge-lorawan-starterkit/tree/docs/main/.github/workflows)
we use there, and at the [resulting documentation](https://azure.github.io/iotedge-lorawan-starterkit/dev/),
including versioning.
