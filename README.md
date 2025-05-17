## Changes from original repo

There was some jank when running `npm run test` which required running these commands before being able to build properly:

-  `npm install --save-dev rewire`
- `cd .\webview-ui\`
- `npm install react-scripts`

I still can't get the test to work properly, but it builds fine via this command:

- `npx vsce package`

No clue why, but it works.  
Probably an issue on my end.

---

This is a work-in-progress test and is not intended to be merged into `main`.

## **The primary change was in `/src/core/prompts/system.ts`.**

As per [this issue comment](https://github.com/cline/cline/issues/1195#issuecomment-2578310501), I noticed that the system prompt was rather large (around 11.5k tokens), which can confuse "weaker" models.

**I removed all mentions of MCP servers and readjusted the system prompt to use a more standard markdown formatting.**

This brings the initial system prompt down to around 6.5k tokens.

**I have also included a "forced" user prompt, which is this:**
```
replace_in_file needs to be formatted exactly like this:

<<<<<<< SEARCH
[exact content to find]
=======
[new content to replace with]
>>>>>>> REPLACE

Every single block of search/replace must be complete or the task will fail. This is very important.

Search only for the specific line you want to change, not an entire block. Code changed with this tool should be limited to one line at a time per search/replace function.

Do not try to search/replace any comments.
```

**Search/replace functionality seems to improved *drastically* due to these changes. At least 95% of my `replace_in_file` calls succeed now (as opposed to all of them failing prior), with only ones above the context window of the model being flaky (but still succeeding more often than not.)**

The model still does not want to follow the "one change per line" request 100% of the time, but that doesn't seem to matter. It helps to promote that suggestion though and seems to push the model towards that output.

This was tested with `mistral-large-latest` via Mistral's free/experimental API, using `https://api.braintrust.dev/v1/proxy`.

I'd imagine it would work with other models as well.

---

I've included a built version in the releases, if you want to try it for yourself without having to build it.

I would highly recommend building it yourself though.

---

## **I believe the system prompt could be improved further and the MCP server calls be reimplemented for "weaker" models.**

I currently do not use MCP servers (I couldn't get them to work on my end), so this change doesn't limit the abilities of the extension for my use cases.

I'd gesture that all of the MCP server prompts (how to create them, how to call them, etc) should be moved outside of the main system prompt and only called when necessary. I need to read up a bit more on Typescript before I attempt that though.

Information about _creating_ MCP servers should only be injected when necessary (and possibly removed from the context after their successful creation).

I've personally found that LLMs can get a bit confused when there's code in the context that is not currently "necessary".

I'll have to look into how things are being called from the system prompt. It looks like it has something to do with the `${cwd.toPosix()}` chunks. Typescript isn't my strongest language.

---

## Other possible changes I'm considering

- Pushing the system prompt file out to an editable configuration file (instead of being built into the extension).
- Forcing the system prompt again when above the context window of the model.

- Forcing search/replace functionality to include indentations (currently search/replace in python is agonizing, requiring multiple calls to fix indentation issues, which can confuse the model).

The third change probably requires backend changes, not system prompt changes. I'll have to do more research into that.

It also might be good to build a separate version of the extension for non-claude models. I know the goal is to have a "one size fits all" extension, but that might not be possible.

Or perhaps have an entirely different system prompt depending on the model...? That seems more feasible.

---

## Other considerations

I also believe the main system prompt could be tightened up in general (just my two cents, of course).
The language seems a bit too "conversational" and I think it could be rephrased to be more "direct".

This line is a good example.
https://github.com/cline/cline/blob/6764a5cdec131e25fd9ef0eebdd993aabb5317f3/src/core/prompts/system.ts#L39

Phrases like:
- "Use this when you need to perform system operations..." 

should be adjusted to something like: 
- "This is used when performing system operations...".

Language like this indicates specific usage and implications of the tool, as opposed to the current phrasing which is more "subjective", requiring the model to infer a bunch of things already stated in the system prompt (what a tool is, how they are used, etc).

The adjustment might seem subtle at first glance, but it's alters the prompt from a "subjective" stance to an "objective" stance. I've personally found that models perform markedly better when prompting in this manner, removing as much wiggle room in interpretation as possible.

---

# Cline (prev. Claude Dev) – \#1 on OpenRouter
>>>>>>> 90d7afc4 (initial changes)

<p align="center">
  <img src="https://media.githubusercontent.com/media/cline/cline/main/assets/docs/demo.gif" width="100%" />
</p>

<div align="center">
<table>
<tbody>
<td align="center">
<a href="https://marketplace.visualstudio.com/items?itemName=saoudrizwan.claude-dev" target="_blank"><strong>Download on VS Marketplace</strong></a>
</td>
<td align="center">
<a href="https://discord.gg/cline" target="_blank"><strong>Discord</strong></a>
</td>
<td align="center">
<a href="https://www.reddit.com/r/cline/" target="_blank"><strong>r/cline</strong></a>
</td>
<td align="center">
<a href="https://github.com/cline/cline/discussions/categories/feature-requests?discussions_q=is%3Aopen+category%3A%22Feature+Requests%22+sort%3Atop" target="_blank"><strong>Feature Requests</strong></a>
</td>
<td align="center">
<a href="https://docs.cline.bot/getting-started/for-new-coders" target="_blank"><strong>Getting Started</strong></a>
</td>
</tbody>
</table>
</div>

Meet Cline, an AI assistant that can use your **CLI** a**N**d **E**ditor.

Thanks to [Claude 3.7 Sonnet's agentic coding capabilities](https://www.anthropic.com/claude/sonnet), Cline can handle complex software development tasks step-by-step. With tools that let him create & edit files, explore large projects, use the browser, and execute terminal commands (after you grant permission), he can assist you in ways that go beyond code completion or tech support. Cline can even use the Model Context Protocol (MCP) to create new tools and extend his own capabilities. While autonomous AI scripts traditionally run in sandboxed environments, this extension provides a human-in-the-loop GUI to approve every file change and terminal command, providing a safe and accessible way to explore the potential of agentic AI.

1. Enter your task and add images to convert mockups into functional apps or fix bugs with screenshots.
2. Cline starts by analyzing your file structure & source code ASTs, running regex searches, and reading relevant files to get up to speed in existing projects. By carefully managing what information is added to context, Cline can provide valuable assistance even for large, complex projects without overwhelming the context window.
3. Once Cline has the information he needs, he can:
    - Create and edit files + monitor linter/compiler errors along the way, letting him proactively fix issues like missing imports and syntax errors on his own.
    - Execute commands directly in your terminal and monitor their output as he works, letting him e.g., react to dev server issues after editing a file.
    - For web development tasks, Cline can launch the site in a headless browser, click, type, scroll, and capture screenshots + console logs, allowing him to fix runtime errors and visual bugs.
4. When a task is completed, Cline will present the result to you with a terminal command like `open -a "Google Chrome" index.html`, which you run with a click of a button.

> [!TIP]
> Use the `CMD/CTRL + Shift + P` shortcut to open the command palette and type "Cline: Open In New Tab" to open the extension as a tab in your editor. This lets you use Cline side-by-side with your file explorer, and see how he changes your workspace more clearly.

---

<img align="right" width="340" src="https://github.com/user-attachments/assets/3cf21e04-7ce9-4d22-a7b9-ba2c595e88a4">

### Use any API and Model

Cline supports API providers like OpenRouter, Anthropic, OpenAI, Google Gemini, AWS Bedrock, Azure, and GCP Vertex. You can also configure any OpenAI compatible API, or use a local model through LM Studio/Ollama. If you're using OpenRouter, the extension fetches their latest model list, allowing you to use the newest models as soon as they're available.

The extension also keeps track of total tokens and API usage cost for the entire task loop and individual requests, keeping you informed of spend every step of the way.

<!-- Transparent pixel to create line break after floating image -->

<img width="2000" height="0" src="https://github.com/user-attachments/assets/ee14e6f7-20b8-4391-9091-8e8e25561929"><br>

<img align="left" width="370" src="https://github.com/user-attachments/assets/81be79a8-1fdb-4028-9129-5fe055e01e76">

### Run Commands in Terminal

Thanks to the new [shell integration updates in VSCode v1.93](https://code.visualstudio.com/updates/v1_93#_terminal-shell-integration-api), Cline can execute commands directly in your terminal and receive the output. This allows him to perform a wide range of tasks, from installing packages and running build scripts to deploying applications, managing databases, and executing tests, all while adapting to your dev environment & toolchain to get the job done right.

For long running processes like dev servers, use the "Proceed While Running" button to let Cline continue in the task while the command runs in the background. As Cline works he’ll be notified of any new terminal output along the way, letting him react to issues that may come up, such as compile-time errors when editing files.

<!-- Transparent pixel to create line break after floating image -->

<img width="2000" height="0" src="https://github.com/user-attachments/assets/ee14e6f7-20b8-4391-9091-8e8e25561929"><br>

<img align="right" width="400" src="https://github.com/user-attachments/assets/c5977833-d9b8-491e-90f9-05f9cd38c588">

### Create and Edit Files

Cline can create and edit files directly in your editor, presenting you a diff view of the changes. You can edit or revert Cline's changes directly in the diff view editor, or provide feedback in chat until you're satisfied with the result. Cline also monitors linter/compiler errors (missing imports, syntax errors, etc.) so he can fix issues that come up along the way on his own.

All changes made by Cline are recorded in your file's Timeline, providing an easy way to track and revert modifications if needed.

<!-- Transparent pixel to create line break after floating image -->

<img width="2000" height="0" src="https://github.com/user-attachments/assets/ee14e6f7-20b8-4391-9091-8e8e25561929"><br>

<img align="left" width="370" src="https://github.com/user-attachments/assets/bc2e85ba-dfeb-4fe6-9942-7cfc4703cbe5">

### Use the Browser

With Claude 3.5 Sonnet's new [Computer Use](https://www.anthropic.com/news/3-5-models-and-computer-use) capability, Cline can launch a browser, click elements, type text, and scroll, capturing screenshots and console logs at each step. This allows for interactive debugging, end-to-end testing, and even general web use! This gives him autonomy to fixing visual bugs and runtime issues without you needing to handhold and copy-pasting error logs yourself.

Try asking Cline to "test the app", and watch as he runs a command like `npm run dev`, launches your locally running dev server in a browser, and performs a series of tests to confirm that everything works. [See a demo here.](https://x.com/sdrzn/status/1850880547825823989)

<!-- Transparent pixel to create line break after floating image -->

<img width="2000" height="0" src="https://github.com/user-attachments/assets/ee14e6f7-20b8-4391-9091-8e8e25561929"><br>

<img align="right" width="350" src="https://github.com/user-attachments/assets/ac0efa14-5c1f-4c26-a42d-9d7c56f5fadd">

### "add a tool that..."

Thanks to the [Model Context Protocol](https://github.com/modelcontextprotocol), Cline can extend his capabilities through custom tools. While you can use [community-made servers](https://github.com/modelcontextprotocol/servers), Cline can instead create and install tools tailored to your specific workflow. Just ask Cline to "add a tool" and he will handle everything, from creating a new MCP server to installing it into the extension. These custom tools then become part of Cline's toolkit, ready to use in future tasks.

-   "add a tool that fetches Jira tickets": Retrieve ticket ACs and put Cline to work
-   "add a tool that manages AWS EC2s": Check server metrics and scale instances up or down
-   "add a tool that pulls the latest PagerDuty incidents": Fetch details and ask Cline to fix bugs

<!-- Transparent pixel to create line break after floating image -->

<img width="2000" height="0" src="https://github.com/user-attachments/assets/ee14e6f7-20b8-4391-9091-8e8e25561929"><br>

<img align="left" width="360" src="https://github.com/user-attachments/assets/7fdf41e6-281a-4b4b-ac19-020b838b6970">

### Add Context

**`@url`:** Paste in a URL for the extension to fetch and convert to markdown, useful when you want to give Cline the latest docs

**`@problems`:** Add workspace errors and warnings ('Problems' panel) for Cline to fix

**`@file`:** Adds a file's contents so you don't have to waste API requests approving read file (+ type to search files)

**`@folder`:** Adds folder's files all at once to speed up your workflow even more

<!-- Transparent pixel to create line break after floating image -->

<img width="2000" height="0" src="https://github.com/user-attachments/assets/ee14e6f7-20b8-4391-9091-8e8e25561929"><br>

<img align="right" width="350" src="https://github.com/user-attachments/assets/140c8606-d3bf-41b9-9a1f-4dbf0d4c90cb">

### Checkpoints: Compare and Restore

As Cline works through a task, the extension takes a snapshot of your workspace at each step. You can use the 'Compare' button to see a diff between the snapshot and your current workspace, and the 'Restore' button to roll back to that point.

For example, when working with a local web server, you can use 'Restore Workspace Only' to quickly test different versions of your app, then use 'Restore Task and Workspace' when you find the version you want to continue building from. This lets you safely explore different approaches without losing progress.

<!-- Transparent pixel to create line break after floating image -->

<img width="2000" height="0" src="https://github.com/user-attachments/assets/ee14e6f7-20b8-4391-9091-8e8e25561929"><br>

## Contributing

To contribute to the project, start with our [Contributing Guide](CONTRIBUTING.md) to learn the basics. You can also join our [Discord](https://discord.gg/cline) to chat with other contributors in the `#contributors` channel. If you're looking for full-time work, check out our open positions on our [careers page](https://cline.bot/join-us)!

<details>
<summary>Local Development Instructions</summary>

1. Clone the repository _(Requires [git-lfs](https://git-lfs.com/))_:
    ```bash
    git clone https://github.com/cline/cline.git
    ```
2. Open the project in VSCode:
    ```bash
    code cline
    ```
3. Install the necessary dependencies for the extension and webview-gui:
    ```bash
    npm run install:all
    ```
4. Launch by pressing `F5` (or `Run`->`Start Debugging`) to open a new VSCode window with the extension loaded. (You may need to install the [esbuild problem matchers extension](https://marketplace.visualstudio.com/items?itemName=connor4312.esbuild-problem-matchers) if you run into issues building the project.)

</details>

<details>
<summary>Creating a Pull Request</summary>

1. Before creating a PR, generate a changeset entry:
    ```bash
    npm run changeset
    ```
   This will prompt you for:
   - Type of change (major, minor, patch)
     - `major` → breaking changes (1.0.0 → 2.0.0)
     - `minor` → new features (1.0.0 → 1.1.0)
     - `patch` → bug fixes (1.0.0 → 1.0.1)
   - Description of your changes

2. Commit your changes and the generated `.changeset` file

3. Push your branch and create a PR on GitHub. Our CI will:
   - Run tests and checks
   - Changesetbot will create a comment showing the version impact
   - When merged to main, changesetbot will create a Version Packages PR
   - When the Version Packages PR is merged, a new release will be published

</details>


## License

[Apache 2.0 © 2025 Cline Bot Inc.](./LICENSE)
