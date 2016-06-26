---
title: Web Developing with Meteor & React
date: 2016-05-25 14:20:11
categories:
- web
tags:
- meteor
- react
- eslint
---

> This will be a full go-through document covering all aspects of developing a web app which including linter config, unit test, isolated component test, Hot Module Reload, etc.

# Tech Stack
## Why Meteor
It provides a homomorphic interface of database to let you process data in the same way from both client and server. With this you will can relieve yourself from the tedious format conventions. (e.g. Object => JSON => Object).

This mechanism is implemented with [Socket.io](http://socket.io) thus the efficiency is guaranteed.

## Why React
It provides modularization and state managing mechanism for rendering process, brings a neat render logic/hierarchy and isolated components with high reusability.

## State Container
Currently `Redux` is popular for managing state changes but it's a little bit heavy. I might consider [react-most](https://github.com/jcouyang/react-most) instead.

# App Setup
> If you're using oh-my-zsh, add 'meteor' to the plugin list to enable zsh completion for meteor command.

For the following chapters, I will create a app called `tasks-manager` as demo.

```
# setup meteor
meteor create tasks-manager
cd tasks-manager
# setup react
npm i -S react react-dom
```

To let React use Meteor's reactive data:
```
npm i -S react-addons-pure-render-mixin
meteor add react-meteor-data
```

# ESLint setup
> For Spacemacs user, to enable the eslint checker, you got to install eslint and all eslint plugins/confis globally

Basically our eslint will apply `airbnb` config as the basis, and adopt `meteor` and `react` specific rules:

```
npm i -D eslint eslint-config-airbnb eslint-plugin-import eslint-plugin-jsx-a11y \
         eslint-plugin-meteor eslint-plugin-react
```

## npm script
use following config in `package.json` to enable command `npm run lint`:

```json
{
  "scripts": {
    "lint": "eslint --ext '.jsx,.js' ."
  }
}
```

## meteor specific
Since we can't install `meteor` via `npm`, the global `eslint` can not resolve the `meteor`'s path. To quite the errors like this, we provide a specific rule to eslint. Here's the full contents of `.eslintrc.yml`:

```yaml
---
  root: true

  env:
    es6: true
    browser: true
    node: true
    meteor: true

  parserOptions:
    ecmaVersion: 6
    sourceType: module

  plugins:
    - meteor
    - react

  extends:
    - airbnb/base
    - plugin:meteor/recommended
    - plugin:react/recommended

  rules:
    import/no-unresolved:
      - 2
      - ignore:
          - ^meteor/
```
