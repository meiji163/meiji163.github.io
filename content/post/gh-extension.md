---
title: "Fun With GitHub CLI Extensions"
date: 2021-12-25
tags: ["github","golang", "gh extensions", "machine learning"]
categories: ["github", "programming", "golang"]
katex: false
---
> And everything else which can be attributed to body presupposes extension, and is only a mode of that which is extended.   
> -- Descartes, _Principles of Philosophy_

## Introduction
`gh` is the official GitHub CLI, maintained in the public [cli/cli](https://github.com/cli/cli) repository.  
Like its predecessor hub[^hub], `gh` wraps common git commands. But `gh` does much more, ultimately aiming to be a general GitHub interface for your terminal.
You can for example run actions with `gh workflow`, publish gists with `gh gist`, or access your codespaces with `gh cs`. If you don't already have it, check out [here](https://cli.github.com/) to get started!
[^hub]: https://github.com/github/hub

Recently, I interned for the GitHub CLI team via the MLH Fellowship[^MLH].
I learned a great deal about software and golang from the senior devs ([@vilmbim](https://github.com/vilmibm), [@mislav](https://github.com/mislav), and [@samcoe](https://github.com/samcoe)).
I was also pleasantly surprised at how easy it was to use and contribute to the CLI. It has great user experience and a ton of features for using GitHub programmatically.
[^MLH]: The MLH Fellowship is basically an internship done in small groups called pods. I thought it was fantastic! Learn more at https://fellowship.mlh.io/

However my favorite part of `gh` is **extensions**, which launched in v2.0. Extensions are programs that you can install, execute, and upgrade -- just like a mini package-manager! 
If there's any functionality you want that falls outside the scope of the core commands, you can find or make an extension that does just that.   

## Using extensions
`gh` installs extensions from GitHub repositories with a "gh-" prefix (e.g. meiji163/gh-coolextension). All that's required is a top-level executable with the same name as the repo. Once you install an extension, it creates a new top-level `gh` command under that name aliased to that executable. Here is an example:

```shell
# install an extension by repo name 
$ gh extension install vilmibm/gh-screensaver  

# execute the extension
$ gh screensaver 

# upgrade the extension
$ gh extensions upgrade screensaver
```

There is already a quickly growing ecosystem of extensions to choose from; when I started in mid-September there were less than 100, and at the time of writing there are [145](https://github.com/topics/gh-extension).

I love not only how useful, but also how creative and fun they are. Here's a few of my favorites: 

- [dlvhdr/gh-prs](https://github.com/dlvhdr/gh-prs): A slick dashboard of all your pull requests, with lots of configurability!

![gh-prs](https://github.com/dlvhdr/gh-prs/blob/8621183574c573e4077360b5027ffea70999b921/demo.gif?raw=true)

- [vilmibm/gh-screensaver](https://github.com/vilmibm/gh-screensaver): Cool and colorful terminal animations (I added the game of life one)
![gh-screensaver](https://user-images.githubusercontent.com/98482/134737341-701d0e7d-476f-4a29-8309-d34b4935c6a3.gif)
![gh firework](https://user-images.githubusercontent.com/98482/134737299-aa306b69-ceb4-49c1-95c8-3582d195250c.gif)

- [yusukebe/gh-markdown-preview](https://github.com/yusukebe/gh-markdown-preview): Renders your local md files with GitHub-flavored markdown in your browser. Super useful for writing readmes.
![gh-markdown](https://user-images.githubusercontent.com/10682/138411417-dd12a831-bacc-4b05-a33d-47d3f6b45483.png)
- [kawarimidoll/gh-graph](https://github.com/kawarimidoll/gh-graph): View your contribution graph in the terminal
![gh-grpah](https://user-images.githubusercontent.com/8146876/131276560-9b97596f-2f21-48ed-a946-498c47d9faf1.png)

## Writing extensions

The CLI team made some tools to help write extensions (nice guys, eh?). Use the `gh extension create` command and you'll be prompted for the name and what kind of extension you want to make. A local git repository will then be created for you with some boilerplate. 

Extensions can be written in any language, but for our purpose languages are split into three categories: interpreted, Golang, and other compiled language. Here are a few key differences between them when writing extensions.

**Interpreted**:
This requires the user to have the interpreter for the language (and any dependencies) installed. For that reason, most are written in bash, but it could be python, ruby, etc.

**Compiled (not golang)**:
This allows you to distribute binaries from your extension repo. 
Precompiled binary extensions are fully supported since gh version 2.3.0. That means if you publish a release with binary assets from your extension repo, `gh extension install` will download the appropriate binary (instead of looking for a top-level executable). 

The binary assets have to be named as "platform-architecture" (e.g. darwin-arm64). Luckily @vilmibm recently made [gh-extension-precompile](https://github.com/cli/gh-extension-precompile), a GitHub action that automates the building and releasing for you. Modify the build script for your language as needed.

**Golang**: The stuff for compiled languages also applies here, but there's even _more_ tools for golang! The CLI team recently open-sourced [go-gh](https://github.com/cli/go-gh), a module for golang that exposes a small subset of cli/cli internals. This includes an HTTP client for both the REST and GraphQL API for GitHub. It also has an `Exec` function that safely shells out to `gh`, and a `CurrentRepository` function. It's still in beta, so there will likely be more functionality to come. See below for an example usage.

### GitHub APIs
One of the main reasons to write gh extensions is that you can easily make authenticated GitHub API calls (although your extension can be completely unrelated to GitHub) since the user will presumably have an auth token for `gh`.

A typical way would be to use the `gh api` command. This command wraps both the REST and GraphQL GitHub API.
You can select format the response using jquery syntax with the `--jq` flag, or (even better) with [golang templates](https://golangdocs.com/templates-in-golang) using the `--template` flag. 
Templates are quite powerful for text formatting. See `gh formatting help` for more details on built-in functions and examples.

For example, here is a script to print a table of a user's starred repos:
```bash 
#!/usr/bin/bash
gh api /users/"$USERNAME"/starred \
    --template '
    {{- tablerow "NAME" "DESCRIPTION" -}}
    {{- range .}}{{- tablerow .name .description -}}{{- end -}}'
```
A little polishing, and this would be a fine bash gh-extension!

If you're using golang, you can use the clients from `go-gh`. Here is how to get the same data as above:
```go
opts := &api.ClientOptions{EnableCache: true}
client, err := gh.RESTClient(opts)
if err != nil {
    log.Fatal(err)
}
response := []struct{ 
    Name string 
    Description string
}{}
err = client.Get(fmt.Sprintf("users/%s/starred",username), &response)
```

You can also use the GraphQL API (somewhat annoyingly, there are features not available in the REST API available in GraphQL and vice versa). Here is a query to get the same data once again:

```graphql
query GetStarredRepos($username: String!) {
  user(login: $username) {
    starredRepositories(first: 100) {
      nodes {
        name
        description
      }
    }
  }
}
```
GitHub has a [great tool](https://docs.github.com/en/graphql/overview/explorer) for exploring their GraphQL schema.
There is also a [GitHub GraphQL library](https://github.com/shurcooL/githubv4) for golang that provides a more concise interface.

### Three more extensions 
To conclude, I want to go over three extensions I wrote during my time on the CLI team. 
They certainly aren't the best quality since I wrote them while learning golang and gh, but hopefully they will serve as examples and perhaps give you ideas for your own extension -- _I'm eager to see what you make!_

**[gh-notify](https://github.com/meiji163/gh-notify):**   
This one is pretty simple. It's a bash script that fetches notifications and pipes it into [fzf](https://github.com/junegunn/fzf). 
If you select a notification linked to a pull request or issue, it will show it with the appropriate `gh` command.
I wrote it for a hackday.

```shell
# show unread notifs
$ gh notify 

# show first 10 of all notifs  
$ gh notify -a -n 10 
```

**[gh-search](https://github.com/meiji163/gh-search):**   
This extension provides access to the search API for repositories. It was a personal tool, as I'm frequently looking for great new projects on GitHub. 

It allows you to filter with `--lang` or `--topic` flags, or simply enter the raw search query in [GitHub syntax](https://docs.github.com/en/search-github/getting-started-with-searching-on-github/understanding-the-search-syntax) with the `-q` flag. 
It then shows you the results in a searchable scrolling menu.

```shell
# search for cli repos with hacktoberfest topic
$ gh search cli --topic=hacktoberfest

# custom search with GitHub syntax
$ gh search -q="org:cli created:>2019-01-01"
```
I wrote it in golang using go-gh and [cobra](https://github.com/spf13/cobra/), a nice library for CLI apps (and the same one used in gh). 

Note that this extension will probably be obsolete'd once `gh` adds search to the core commands (ref: [cli#4834](https://github.com/cli/cli/issues/4834)).

**[gh-spam](https://github.com/meiji163/gh-spam):**   
This extension is a simple classifier for spam issues (again in golang). I wrote it on the last hackday (they have a lot of hackdays at GitHub, which is great). During the internship I noticed a fairly large volume of spam issues on the cli/cli repo that had to be closed manually. 
As an ML-inclined fellow, I thought I would take a crack at automating this. 

To use it, you first download a labeled dataset of issues from a target repo (this only really works if you have a large repo). Then you train a classifier on it, which by default is a [random forest](https://en.wikipedia.org/wiki/Random_forest).

```shell
# download data from cli/cli repo 
$ gh-spam download -R cli/cli

# classify an issue 
$ gh-spam classify -R cli/cli 4894
#4894: spam

# classify multiple issues
$ gh-spam classify -R cli/cli 4913 4907 4906 4894
#4913: not spam
#4907: not spam
#4906: not spam
#4894: spam
```

Rather than do any fancy NLP on the issue content, the classifier inputs (aka features) are based primarily on the "reputation" of the author, i.e. number of contributions, association with the target repo, age of account, etc. 
I also took into account issues which just post one of the issue templates without substantive change by using a string-matching score.
This approach turns out to be sufficiently accurate (at least for the cli/cli repo).

Making this extension was particularly interesting, because I previously only used python for machine learning. I looked around for ML libraries in golang, and ended up using [golearn](https://github.com/sjwhitworth/golearn). It has a nice API similar to scikit-learn, but in comparison it is pretty awkward for handling data, and has less clear documentation[^docs].
[^docs]: The documentation complaint is a "golang vs. python" complaint more than a "golearn vs. sklearn" complaint.

## Further Resources 
Here are a few more resources you may find useful.
- [official GitHub guide](https://docs.github.com/en/github-cli/github-cli/creating-github-cli-extensions) for extensions
- [precompiled extension demo](https://www.youtube.com/watch?v=tzK3ZuiEjcA&feature=youtu.be&ab_channel=vilmibm) by @vilmibm
- [GitHub blogpost](https://github.blog/2021-08-24-github-cli-2-0-includes-extensions/) on extensions launch 
- [manual](https://cli.github.com/manual/) for gh 
