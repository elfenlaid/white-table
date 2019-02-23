---
layout: post
title: "Fine graining Elixir tests in VSCode"
categories: elixir vscode
---

Most part of my programming years I was cruising on a board of IDE. Beside short dwellings into an old school editors like Emacs or Vim I kept loyal to IDEs. IDE are caring, capable and well, integrated into process.
But everything starts to fell apart when you need to mess around with web applications. So VSCode and it's plugins system comes to rescue.

When it comes to build tools, Elixir has a really decent one. `mix` is one of the best build tools I have ever worked with. Without further ado, lets introduce `mix` into VSCode tasks.

Basic VSCode Elixir setup includes [JakeBecker's Elixir Language Server](https://github.com/JakeBecker/vscode-elixir-ls) plugin. Which can format code, inline build errors and much more.

Also plugin somewhat provides ability to interact with `mix test`, but it requires a bit of tuning.

## Setup Tasks

VSCode can interact with external tooling via so called tasks. Tasks is nothing else but `json` config file which lives at `.vscode/tasks.json`. Here's a basic tests setup.

```json
{
  // See https://go.microsoft.com/fwlink/?LinkId=733558
  // for the documentation about the tasks.json format
  "version": "2.0.0",
  "tasks": [
    {
      "label": "mix test",
      "type": "shell",
      "command": "mix",
      "args": [
        "test",
        "--color",
        "--trace"
      ],
      "options": {
        "cwd": "${workspaceRoot}",
        "requireFiles": [
          "test/**/test_helper.exs",
          "test/**/*_test.exs"
        ],
      },
      "problemMatcher": "$mixTestFailure"
    }
  ]
}
```

Now we can use `> Tasks: Run Tasks` from command pallet, pick `mix test` and see test results. Cool.

Let's do a quick glance at task:

- `"command": "mix"` use terminal `mix` command with args:
  - `test` - launch project tests
  - `--color` - everyone loves colored output, if you don't just remove it
  - `--trace` - provides detailed test output
- `"options":` - setup command launch options like `pwd` and preconditions to run it
- `"problemMatcher": "$mixTestFailure"` - helps to inline `mix test` output into VSCode UI. Matcher provided by  [JakeBecker's ElixirLS](https://github.com/JakeBecker/vscode-elixir-ls) plugin. Matcher itself is nothing more than bunch of [regexes](https://github.com/JakeBecker/vscode-elixir-ls/blob/master/package.json#L306-L325).

Now we know what all this task things were about.

But `mix` can do much more. Like launch specific test or all tests from one file, or launch only tagged tests. Lets explore that a bit:

```json
{
  "label": "mix test item",
  "type": "shell",
  "command": "mix",
  "args": [
    "test",
    "${relativeFile}:${lineNumber}",
    "--color",
    "--trace"
  ],
  "options": {
    "cwd": "${workspaceRoot}",
    "requireFiles": [
      "test/**/test_helper.exs",
      "test/**/*_test.exs"
    ],
  },
  "problemMatcher": "$mixTestFailure"
}
```

Task should look familiar to previous one apart from `"${relativeFile}:${lineNumber}"` line, which tells `mix` to run test from cursor line.

Swap it to `"${relativeFile}"` and now we can run all tests from current file. Add `--only debug` option to run only tagged `@tag :debug` tests. Awesome.

## Setup Shortcuts

So, we have an ability to run tests in all imaginable ways, but launching them from ``> Tasks: Run Tasks` menu quickly becomes mundane. One thing we can do to straightforward process is to provide task shortcut.

Use `> Preferences: Open Keyboard Shortcuts (JSON)` to jump to keyboard shortcuts file and assign task key combination:

```json
  {
    "key": "alt+t",
    "command": "workbench.action.tasks.runTask",
    "args": "mix all"
  },
  {
    "key": "alt+shift+t",
    "command": "workbench.action.tasks.runTask",
    "args": "mix test item"
  }
```

Here we're specifying which task to run from `tasks.json` file via passing task's `"label"` value into `"args"` value.

Just look at that little shortcut that we have now :)

## Summary

I hope that post has cleared situation around VSCode tasks and simplifies your daily test routine. Happy testing and keep it up :)