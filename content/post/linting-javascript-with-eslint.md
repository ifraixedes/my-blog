+++
categories = ["Software Development"]
description = "Know why linting your javascript code is so important and learn how to lint your javascript code with the power ESLint"
tags = ["javascript", "programming", "best-practices"]
date = "2016-06-11T23:12:07Z"
title = "Linting Javascript with ESLint"
social_image = "https://s-media-cache-ak0.pinimg.com/originals/41/0a/14/410a14e6f68619b3ebfc10abf1f611a9.png"
+++

## Linting Javascript

Javascript as you well known, is a dynamic and untyped language; characteristics that have their advantages but which also have a few drawbacks which lead to easily introduce trivial bugs in the implementations.

For the good or the bad of the Javascript language, nowadays is quite difficult to escape from it because any website with a good UX requires quite certain amount of Javascript code, moreover, if we're talking about of Single Page Application the numbers of lines of code of Javascript increase drastically.

On the other hand, the big popularity of NodeJS has made Javascript a server side programming language, however if you aren't supporter of it, you can scape of it because on server side there are many options, without having the drawbacks that in Web frontend code you can face when choosing one of the transpilers to escape of it.

If you are supporter of Javascript or you may not, but you don't have the chance to replace it with anything else, then you must use a linter. Linters aren't new in Javascript, they haven't been for awhile, from the first famous one {{<ext-link "JSLint" "https://github.com/douglascrockford/JSLint">}} created by Douglas Crockford, its fork {{<ext-link "JSHint" "https://github.com/jshint/jshint">}} and {{<ext-link "ESLint" "http://eslint.org/">}}, between some others.

A Javascript linter make our codebase more maintainable, because it does a static analysis of the codebase to enforces syntax rules for formatting, which is quite important to keep the code base consistent across the team members, and, even more important, to detect potential errors, without executing the code.


## The ESLint benefits

ESlint marked a difference between the previous linters, reason why it looks the best option today, if you are not looking for the fastest one.

The main difference between ESLint and its predecessors are:

* It uses an AST to evaluate patterns in code.
* It is completely pluggable, every single rule is a plugin and you can add more at runtime.

Let's go one by one to see how they benefit to us.

ESLint uses Espree to construct an AST of the codebase which is used to evaluate all the rules; this is a big difference in how others linters operates (for example JSHint), which they only do a lexical analysis, being faster but at the same time they are limited to check only a set of rules, because some of them are not possible to do it without constructing the AST, for example, "Require Consistent This"

The drawback of doing so it's that the the linter is slower, but if the number of rules used aren't huge, it's totally acceptable and the benefits are worth.

Being pluggable allow us to enable and disable any rule, so we can configure to fit our team needs or just set in place in current implementations where some rules are constrained to it than the option that we would desire. There are several plugins available, some of them allows to uses lint code for some frameworks, for example, {{<ext-link "ES7 features" "https://github.com/babel/babel-eslint">}}, {{<ext-link "AngularJS" "https://github.com/Gillespie59/eslint-plugin-angular">}}, {{<ext-link "flow type annotations" "https://github.com/zertosh/eslint-plugin-flow-vars">}}, etc.

ESLint also allows to extend configuration files, so we can extend set of rules a defined file and add or override certain rules to fit in our configuration file using the ones listed in the {{<ext-link "its long list" "http://eslint.org/docs/rules/">}}, using new ones which has been created  by the community or implement our ones if there isn't any which fits what we need.


## ESLint Setup

Setting up ESLint in your Javascript project is pretty straightforward, as it's available as an {{<ext-link "NPM" "https://www.npmjs.com">}} package.

Assuming that you have installed, an add to your PATH, {{<ext-link "NodeJS" "https://nodejs.org">}} which brings NPM, let's see how to proceed to setup ESLint and configure it.

* Go to the folder where you have your JS project;
* For simplicity, we install ESLint globally in the machine with `npm install -g eslint`; however it's advisable to install it locally to the project to have more control of the version to use, which is very helpful if you have other projects than use an old version of ESLint or you start now with this but in future project you end up using a newer version, to do so, make sure that you have a `package.json` in the project or create one with `npm init` and after execute `npm install --save-dev eslint`.
* Create a minimal ESLint configuration file executing `eslint --init`; reply the question that ESLint asks in order to set the minimum rules which fit to your code style, if you use ES6 or JSX, define the environment where JS executes, the Browser or NodeJS and choose the format that you want between JS, JSON or YAML to store the configuration; you also have the option to set this configuration in `package.json` file with the property `eslintConfig`.
* Right now you should see that you have a file in your folder named `.eslintrc.*` where `*` is the extension used for the format that you have chosen. It's worth to mention that JSON & YAML formats support JS commends to make them more human-friendly, however they aren't supported in `package.json` file.

The basic file which has been generated probably contains only a few properties which ESLint support; probably you'll have the `env` which define the environment where code is executed, which module system is used and ES6 version is used. To see all the options that you can set in this property check {{<ext-link "Specifying Environments in the documentation" "http://eslint.org/docs/user-guide/configuring#specifying-environments">}}.

This basic file extends the rules from the `eslint:recommended` configuration as you can see in the value of `extends` property, but just leaved it this for now and see the `rules` property.

`rules` property contains the rules that we want to use to lint our code base; as commented above, if we extend them from another then we can set ones that the base version doesn't set to cover parts that the base configuration doesn't and set the same one than base configuration has in order to override the rule configuration.

If you never seen how the `rules` are specified in ESLint, you may thing that the format it's odd; they use the name of the rule as a property name, which is very usual, and the value can be an array; they value is the part that it can looks odd because, you can see sometimes an string, an integer or an array, but nothing is weird when you know what they define.

The rule values can have the next string values:
* "off": The rules doesn't apply. By default ESLint doesn't apply any rule so all they are off, however it's useful to set a rule to off when you extends from a base configuration and you want to disable some of the rules.
* "warn": The rules applies with a warning; when ESLint finds parts of the code base where a rule set as a warning, it shows the output message as warning and it ignores them in terms of the exit code, hence ESLint exits with 0 if only rules set as a warning aren't respected in the code base.
* "error": The rules set as error, are show in the output message an an error and if any of them are found during the linting process, then ESLint exit with 1.

Ok, but what an integer means as value then? they can only be 0, 1, 2 and they have the same meaning than above, 0 = "off", 1 ="warn" and 2 ="error"

The last form of value for a rule is an array. Arrays are used when the rule accepts options, the first value of the array is a string or integer as defined above and the rest depends of the rule, for example the {{<ext-link "rule \"quote-props\"" "http://eslint.org/docs/rules/quote-props">}} allows to define if we allow to surround the object properties by quotes, the first option (second position in the array) is to set how ESLint should interpret the rules and depending of which value is selected as a first option, it is possible to define the behavior, which is set as a sub-option (third position in the array).

Let's see the next simple snipped which shows an example of a basic `.eslintrc.json` file, using the different value formats commented

{{<highlight json>}}
{
  "env": {
    "browser": true,
    "commonjs": true,
    "es6": true
  },
  "extends": "eslint:recommended",
  "rules": {
    "indent": ["error", 2],
    "linebreak-style": [2, "unix"],
    "quotes": 2,
    "semi": [1, "never"],
    "quote-props": ["warn", "consistent-as-needed", { "keywords": true }],
    "brace-style": "off"
  }
}
{{</highlight>}}

ESLint has other setting for more advanced tweaking, for example, options to set to the parser, parser to use and so on; {{<ext-link "it also has plugins which allow to define, not just rules and environments but how to process non pure JS files" "http://eslint.org/docs/developer-guide/working-with-plugins">}}.

On the other hand, ESLint also allows {{<ext-link "to ignore files and directories as git does, through a `.eslintignore` file" "http://eslint.org/docs/user-guide/configuring#ignoring-files-and-directories">}} and {{<ext-link "to define settings to apply only in parts of the source code using comments" "http://eslint.org/docs/user-guide/configuring#disabling-rules-with-inline-comments">}}, for example `/* eslint-disable semi */` will disable the rule from the line of this comment until the end of the file.


## Running ESLint

Once you setup ESLint in you project you have to execute it.

The command line interface of ESLint is as usual as any other command line; you can see the help information with `-h` option an see all the options that you can pass it.

If you have set the configuration in any of the files which ESLint detect, then you can can run the command an pass the path to sources files or folder where the source files are; the paths are defined with globs so you can set a root folder and it recursively will check all the sources files contained in the folder and the children folders, for example `eslint src/**/*` will check all the source files in src and all the folder subtree inside of it.

{{<ext-link "With `-c` option you can specify a specific configuration file" "http://eslint.org/docs/user-guide/configuring#using-configuration-files">}}; this option is quite useful when you want different configurations for different parts of a project;  however if you are using the configurations files or `package.json` property mentioned in the previous version, it will use them automatically and moreover you can have several of them in different subfolders of the root filesystem to walk specifying different configurations for the sources contained in the subtree and it will use the closest one found to the analyzed source file, see an example of this case in the {{<ext-link "documentation" "http://eslint.org/docs/user-guide/configuring#configuration-cascading-and-hierarchy">}}

Another important option to use to speed up the linting process is `--cache`; enabling it ESLint store information about the processed files in order to only check the ones that have been modified in the following runs. The cache file is store in the root directory with the name `.eslintcache` but you can configure the location with the `--cache-location` option.

ESLint documentation has a dedicated page for {{<ext-link "its command line interface" "http://eslint.org/docs/user-guide/command-line-interface">}}, check it if you want to know all the other options which accepts.


That's all for today folks
