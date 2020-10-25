# Packer {#packer-overview}

In the previous chapter a simple shiny application using NPM and webpack was put together. Hopefully it hinted at some of the powerful things webpack can do but also revealed a downside: the overhead in merely creating the project. Not only these operations have to be repeated for every project, the projects themselves change: the configuration of webpack used previously will work for other shiny applications but will not for R packages and golem [@R-golem] applications as the output files and how they are used will drastically change.

In this chapter we discover the [packer](https://github.com/JohnCoene/packer) [@R-packer] R package which provides many convenience functions to create and manage R projects that make use of webpack and NPM.

```r
install.packages("packer")
```

## Principles {#packer-principles}

There are a few principles that the packer package follows strictly.

1. It only aspires to become a specialised usethis for working with JavaScript and R, it takes inspiration from other packages such as htmlwidgets, usethis, and devtools.
2. It never becomes a dependency to what you create. It's in a sense very much like an NPM "developer" dependency, it's used to develop the project but does not bring any additional overhead to what you're building.
3. It should break the mission of webpack which is to build more robust JavaScript code. Therefore packer only builds on top of already strict R projects, namely packages (where golem packages can be used to create shiny applications).

## Scaffolds {#packer-scaffolds}

Packer is comprised of surprisingly few functions, the most important ones are in the `scaffold` family. The term scaffold was shamelessly taken from the htmlwidgets package which features the function `scaffoldWidget` that was used in this book. The idea of scaffolds in packer are very similar to the `scaffoldWidget` function: it sets up the basic structure for a project.

Whist htmlwidgets only allow creating scaffolds for widgets packer allows creating scaffold various types of projects that involve JavaScript, namely:

- Widgets with `scaffold_widget`
- Shiny inputs with `scaffold_input`
- Shiny outputs with `scaffold_output`
- Shiny extensions with `scaffold_extension`
- Golem applications with `scaffold_golem`

This gives a few powerful function that (almost) all of the job of correctly setting up webpack for the project you are working on. These will configure the project as necessary depending on the scaffold you lay down and the context, whether it is a bare package, a golem application, a package with an existing scaffold, etc.

With some variation that will be explored in the coming sections, packer's `scaffold` functions generally do the following:

- Initialises npm with `npm init -y`
- Install webpack and its CLI with `npm install webpack webpack-cli --save-dev`
- Creates three webpack configuration files: `webpack.common.js`, `webpack.prod.js`, and `webpack.dev.js`
- Creates the `srcjs` directory for the JavaScript source code.
- Creates basic JavaScript files within the `srcjs` directory, e.g.: `index.js`
- Adds the necessary NPM scripts to `package.json`
- Adds all relevant files to the `.Rbuildignore` and `.gitignore` files
- Adds relevant dependencies to the `DESCRIPTION`, e.g.: `shiny` when scaffolding an input
- Finally, opens interesting files to develop the project in the IDE

In the following sections we cover each of the scaffold individually in greater detail.

## Widgets {#packer-widgets}

The widget scaffold, like all other scaffolds must be run from within the root of a package. To demonstrate we'll write a widget for the [countup](https://github.com/inorganik/countUp.js/) library that allows animating numbers.

```r
usethis::create_package("countup")
```

From the root of the package we scaffold the widget with `scaffold_widget`, which prints out some information on what packer exactly does. 

```r
packer::scaffold_widget("countup")
```

```
── Scaffolding widget ──────────────────────────────── countup ── 
✔ Bare widget setup
✔ Created srcjs directory
✔ Initialiased npm
✔ webpack, webpack-cli, webpack-merge installed with scope dev
✔ Created srcjs/config directory
✔ Created webpack config files
✔ Created srcjs/modules directory
✔ Created srcjs/widgets directory
✔ Created srcjs/index.js
✔ Moved bare widget to srcjs
✔ Added npm scripts

── Adding files to '.gitignore' and '.Rbuildignore' ──

✔ Setting active project to '/Projects/countup'
✔ Adding '^srcjs$' to '.Rbuildignore'
✔ Adding '^node_modules$' to '.Rbuildignore'
✔ Adding '^package\\.json$' to '.Rbuildignore'
✔ Adding '^package-lock\\.json$' to '.Rbuildignore'
✔ Adding '^webpack\\.dev\\.js$' to '.Rbuildignore'
✔ Adding '^webpack\\.prod\\.js$' to '.Rbuildignore'
✔ Adding '^webpack\\.common\\.js$' to '.Rbuildignore'
✔ Adding 'node_modules' to '.gitignore'

── Adding packages to Imports ──

✔ Adding 'htmlwidgets' to Imports field in DESCRIPTION
● Refer to functions with `htmlwidgets::fun()`

── Scaffold built ──

ℹ Run `bundle` to build the JavaScript files
```

Importantly, it runs `htmlwidgets::scaffoldWidget` internally.