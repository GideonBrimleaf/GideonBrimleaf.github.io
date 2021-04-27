---
layout: post
title: Deploying Rails to Heroku via CircleCI
author: Gideon Brimleaf
postHero: /assets/images/elixir-intellij-logo.png
description: How to deploy a rails app automatically to Heroku with CircleCI for Integration
---

1. Rails app - make sure it's using PostGreSQL.  Making `bundle exec rails new -d postgresql` and stealing the `config database.yml` file, changing the names to match your project. Add `gem 'pg'` to your `Gemfile` and bundle install.
2. Deploy to Heroku - https://devcenter.heroku.com/articles/getting-started-with-rails6
3. Sign up to CircleCI with Github - link your accounts
4. In CircleCI find your project and click 'Set up Project'
5. Grab the auto-generated `config.yml` file.  Don't do the automatic commit.
6. Create a `.circleci` directory in the project root.  Add the `config.yml` file
7. Add the initial build step.  Note the orb number for both node and ruby and the ruby image.  The latter needs to match the ruby version you are running.

```
version: 2.1
orbs:
  ruby: circleci/ruby@1.1.2
  node: circleci/node@2

jobs:
  build:
    docker:
      - image: cimg/ruby:2.7.2-node
    executor: ruby/default
    steps:
      - checkout
      - ruby/install-deps
      - node/install-packages:
          pkg-manager: yarn
          cache-key: "yarn.lock"

workflows:
  build_and_test:
    jobs:
      - build
      - test:
          requires:
            - build
```

8. Committing this and pushing it up will automatically trigger a build pipeline.  Not very interesting as it is just building.  Would be good to add the rubocop linter (if you have this installed):
```
steps:
      - checkout
      - ruby/install-deps
      - node/install-packages:
          pkg-manager: yarn
          cache-key: "yarn.lock"
      - ruby/rubocop-check
```

9. Making the tests run is a bit more difficult - we need a db setup. Note the postgres version required

```
test:
    docker:
      - image: 'cimg/ruby:2.7.2-node'
      - environment:
          POSTGRES_DB: it_is_hot_pal_test
          POSTGRES_PASSWORD: ''
          POSTGRES_USER: it_is_hot_pal_test
        image: 'circleci/postgres:9.5-alpine'
    environment:
      BUNDLE_JOBS: '3'
      BUNDLE_RETRY: '3'
      PGHOST: 127.0.0.1
      PGPASSWORD: ''
      PGUSER: it_is_hot_pal_test
      RAILS_ENV: test
    parallelism: 3
    steps:
      - checkout
      - ruby/install-deps
      - node/install-packages:
          cache-key: yarn.lock
          pkg-manager: yarn
      - run:
          command: 'dockerize -wait tcp://localhost:5432 -timeout 1m'
          name: Wait for DB
      - run:
          command: 'bundle exec rails db:setup --trace'
          name: Database setup
      - ruby/rspec-test

workflows:
  build_and_test:
    jobs:
      - build
      - test:
          requires:
            - build
```
 10. Now when you push, CircleCI will run the build job then the test job.
 11. How can we make it so that it deploys to Heroku on a successful merge?  The first step is to set up [GitHub Checks](https://circleci.com/docs/2.0/enable-checks/).  We can then test this by checking out a new branch, making a small change and pushing to GitHub.  Raising a new PR shows a checks tab which will show if the CI pipeline was successful or not.
 12. Now we can automatically deploy to Heroku from Github with our CI pipeline in place - [Heroku docs](https://devcenter.heroku.com/articles/github-integration). There is an option to delay the deployment until the CI checks have completed, make sure this is ticked.
 13. Merge your PR into main, watch the CI pipeline kick off and a fresh deployment straight after!