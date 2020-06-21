# Notes about using Mkdocs, Docker, Travis and Github Pages

Notes from learning how to create this note taking website hosted on Github Pages, using the Mkdocs. The deployment is automated using Docker and Travis.
Check out the [repo](https://github.com/aluxh/pen-my-notes)

## To begin

- Create a folder to store your notes.
- The folder structure should look like:

```bash
note
  |_ docs (folder to store all your notes)
  |_ .gitignore
  |_ .travis.yml
  |_ mkdocs.yml
  |_ README.md
```

#### In the .gitignore

you should indicate git should ignore the folder **site**. Because, this is where your content will be built and published, and you shouldn't include it into your repo.

```.gitignore
\site\
```

#### In mkdocs.yml

you can keep it simple in the beginning:

```mkdocs.yml
theme: material
```

#### In travis.yml

We will be using [squidfunk/mkdocs-material](https://hub.docker.com/r/squidfunk/mkdocs-material/) docker to help build the content. To know more about Mkdocs Material theme, you can check out the [repo](https://github.com/squidfunk/mkdocs-material)

- The below script will also use `README.md` as the index page for the site too.
- Running the docker `build --clean` will build the content into the `docs\site\` folder.
- In `deploy` section, we will push the content in `site` to the `gh-pages` branch.
- `cleanup` need to set as `true`, else the published content will be deleted.

```travis
os: linux
language: minimal

services:
  - docker

before_deploy:
  - docker pull squidfunk/mkdocs-material
  - git branch gh-pages
  - cp README.md docs/index.md
  - docker run -ti -v `pwd`:/docs squidfunk/mkdocs-material build --clean
  - ls site

deploy:
  provider: pages
  cleanup: true
  token: $GITHUB_TOKEN
  local_dir: site
  keep_history: true
  verbose: true
  on:
    branch: master
```
