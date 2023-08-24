---
title: "Creating a CLI Utility Belt"
date: "2023-08-24"
draft: false
tags: ["typescript", "javascript", "node", "npm"]
slug: "cli-utility-belt"
---

Hello there, readers!

As a keyboard addict myself, I can't stay long away from the computer. In fact, I'm on the second half of my paternity leave and eager to build something and have a break from endless dirty diapers.

## The Idea

So here's the simple idea: I want to create a [CLI](https://en.wikipedia.org/wiki/Command-line_interface) program to provide utilities that I use on a daily basis as a software engineer. The language I'm going to be using is Typescript. The utilities, or features, I want to include for the first version are:

1. Encoding/decoding Base64 strings
2. Pretty printing/minifying JSON payloads
3. Diffing two files

These have an OK scope to start with, but the intention here is to have an ever growing set of utilities that will be ammended to the program, on the long run.

> But, Gabe... Typescript? Really? I know you aren't fond of this ecosystem. Why then?

Besides the CLI interface which will allow me to integrate it to my workflow that is heavily terminal based, I want to create this "package" with well defined interfaces, so the utilities can also be consumed by a web UI. And you know... when it comes to the browser, we must to fallback to Typescript/Javascript.

> And what about WASM?

Well, I haven't delved into that, yet. Actually, this could be something nice to try in the future.

> Isn't this toolkit going against the minimalist UNIX philosophy of having one particular tool for one particular job?

Yes.

## The Planning

### Naming

There's a saying by Phil Karlton in the programming world that states that "there are only two hard things in Computer Science: cache invalidation and naming things". I'd say off by one errors are also hard, although, that is a different conversation.

I don't believe this initiative will envolve dealing with caching at all, but I needed a name for this toolkit and after some (not really deep) thinkering I decided to go with `ubt`. Some reasons behind this choice are:

- I'm unaware of another tool called `ubt`, although there's probably one already existent. (Note that if you're the creator of the genuine `ubt`, please forgive my lack of originality!)
- If you try hard enough, this is a mneumonic for "utility belt"

### Ergonomics

By having just a few letters, this can be quickly invoked on the command prompt and has potential to build some muscle memory for the touch typists.

Moreover, while in the CLI realm, I want each utility to be invoked as a sub-command, so if we consider the set of 3 utilities I [listed above](#the-idea), those can be `b64`, `json` and `diff`.

As a matter of fact, it is possible to divide these sub-commands into distinct categories:

1. Utilities that receive one parameter: `b64`, `json`
2. Utility that receives two parameters: `diff`

For the first case, I want the program to read the input from the [stdin](https://en.wikipedia.org/wiki/Standard_streams#Standard_input_(stdin)), so it can play nicer with other tools. For instance:

```shell
$ echo "string to be encoded in base64" | ubt b64 -e
```
Whereas, for the second case, we will be using positional arguments:

```shell
$ ubt diff <file1> <file2>
```
## The Scaffolding

To begin with, it's worth mentioning that [working on a already exsiting Typescript codebase and setting one up from scratch are two different beasts](https://www.linkedin.com/pulse/greenfield-vs-brownfield-giorgi-bastos/). In my opinion the Javascript ecosystem is tremendously fragmented if compared to other ecosystems, such as Go or .NET — there are multiple ways and tools to achieve anything you could possibly imagine. There's even an ongoing [joke about the number of days since the last Javascript framework](https://dayssincelastjavascriptframework.com/)! But that isn'nt a problem per se, as long as you're familiar with the options available. 

With that being said, I have to admit I had loads of fun setting the project up.

For the tooling I chose to go with:

- [Yarn](https://yarnpkg.com/) for dependency management
- [ESLint](https://eslint.org/) for linting
- [Prettier](https://prettier.io/) for formatting
- [Jest](https://jestjs.io/pt-BR/) for testing
- [Husky](https://typicode.github.io/husky/) for nifty git hooks

Besides these, there's also the Typescript transpilation and its configurations.

## The Execution

We have all the bells and whistles in place, but yet no code. Let's look at how things were built and why they were built this way.

### Project Structure

I opted for a "as flat as possiblie" directory structure. Given the few constraints for this problem domain, I didn't want to over engineer the project.

Here's what the `src` directory with omitted test files looks like:

```shell
$ tree src -I '*.test.ts'
src
├── base64.ts
├── cli.ts
├── diff
│   ├── index.ts
│   └── stringFormatter.ts
├── index.ts
└── json.ts

2 directories, 6 files
```
The `index.ts` is the program entry point. It imports the command line parser factory that is defined in `cli.ts`, creates an instance of it for parsing the provided arguments.

The `cli.ts` is also responsible for gluing together the functionality that is defined in `base64.ts`, `diff` and `json.ts`.

The only reason `diff` differs (pun intended!) from the other functionalities by having a folder instead of a single file, is because it was built in a way that allows for [future extensions](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle). Suppose that we want this package being used behind the scenes by a web UI — we could create a `htmlFormatter.ts`, or something in this line, and inherit from the constructs that are exported on the `diff/index.ts` to allow for [DRYness](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).

### Test Driven Development

I decided to operate in a [TDD](https://en.wikipedia.org/wiki/Test-driven_development) fashion for this little project.

Besides all of the evangelism that people do about TDD on the internet, for me, the most underrated benefit of it is the sense of progress obtained from the quick feedback you get. You see the tests fail, implement a little functionality, run the tests again and then see them pass. When working solo on a project, this is essential.

The second best side effect of this technique is that you tend to end up with better defined interfaces, as you are forced to, _a priori_, write testing code that acts as the software artifact that is going to be consuming and invoking the functions/methods under test.

However, for TDD to be effectively done, you need to have solid and well defined requirements. Ideally, you'd only start implementing it after modeling the system or going through a prototyping stage, otherwise, you'd end up having to spend time figuring out how your components are going to interact with one another and having a ton of rework fixing tests as you move towards the ideal solution.

## The Result

<script async id="asciicast-O3Q731849e7MSwUm1FmGBQ0dG" src="https://asciinema.org/a/O3Q731849e7MSwUm1FmGBQ0dG.js"></script>

## Consumption

In case you want to use this tool, you'll need a Javascript package manager. 

For `yarn`, you'll have to run the following command:

```shell
$ yarn global add ubt
```
In case you're using `npm`, you'll have to run the following command:

```shell
$ npm install -g ubt
```

That should add the `ubt` script as an executable to whatever your package manager's installation directory is. Make sure that directory is available on your `$PATH`, so you can invoke the command from any directory on your file system.

## References

- [Github Repository](https://github.com/ewilazarus/ubt)
- [Npm Package](https://www.npmjs.com/package/ubt)

-----

Thanks for reading! I had a great time writing this blog post and creating this tool. 

Stay tuned for more!