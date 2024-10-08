<div align="center">

<a href="https://crates.io/crates/devai"><img src="https://img.shields.io/crates/v/devai.svg" /></a>
<a href="https://github.com/jeremychone/rust-devai"><img alt="Static Badge" src="https://img.shields.io/badge/GitHub-Repo?color=%23336699"></a>
<a href="https://www.youtube.com/watch?v=DSuvkCHdD5I&list=PL7r-PXl6ZPcBcLsBdBABOFUuLziNyigqj"><img alt="Static Badge" src="https://img.shields.io/badge/YouTube_devai_Intro-Video?style=flat&logo=youtube&color=%23ff0000"></a>

</div>

# **devai** - **Command Agent File Runner**

```sh

# install
cargo install devai

# Will fix all code comment in all matching file
devai run proof-comments -f "./src/m*.rs" 

# How: It will run the installed Command Agent file ".devai/defaults/proof-comments.md" on all source files matching "./src/m*.rs"

# IMPORTANT: Make sure everything is committed before usage.
```

ONE **Command Agent Markdown File** that defines the full agent flow:
- `items` get expanded from the `-f` file matches (more ways to generate items later).
- `-> Data` **scripting** for getting full control over what data to put in the context.
- `-> Instruction` templating (Handlebars) to have full control over the prompt layout.
- `-> Output` **scripting** to get full control over how to manage the AI output.

`Data`, `Instruction`, `Output` (and more later) are all defined in a single file (see below), which is called the **Command Agent File** 

Supports all models/providers supported by the [genai crate](https://crates.io/crates/genai) (see below for more information).

You can customize the model and concurrency in `.devai/config.toml`.


**IMPORTANT**: Make sure to run this command line when everything is committed, so that overwritten files can be reverted easily.

STILL IN HEAVY DEVELOPMENT... But it's starting to get pretty cool.

_P.S. If possible, try to refrain from publishing `devai-custom` type crates, as this might be more confusing than helpful. However, any other name is great._

## API Keys

**devai** uses the [genai crate](https://crates.io/crates/genai), and therefore the simplest way to provide the API keys for each provider is via environment variables in the terminal when running devai.

Here are the environment variable names used:

```
OPENAI_API_KEY
ANTHROPIC_API_KEY
MODEL_GEMINI
GEMINI_API_KEY
GROQ_API_KEY
COHERE_API_KEY
```

## Usage & Concept

Usage: `devai run proof-comments -f "./src/main.rs"`

(or have any glob like `-f "./src/**/*.rs"` )
- This will initialize the `.devai/defaults` folder with the "Command Agent Markdown" `proof-comments.md` (see [.devai/defaults/proof-comments.md`](./_base/agents/proof-comments.md)) and run it with genai as follows: 
    - `-f "./src/**/*.rs"`: The `-f` command line argument takes a glob and will create an "item" for each file, which can then be accessed in the `# Data` scripting section.
    - `# Data`, which contains a ```rhai``` block that will get executed with the `item` value (the file reference in our example above).
        - With `rhai`, there are some utility functions to list files, load file content, and such that can then be used in the instruction section. 
    - `# Instruction`, which is a Handlebars template section, has access to `item` as well as the output of the `# Data` section, accessible as the `data` variable. 
        - This will be sent to the AI.
    - `# Output`, which now executes another ```rhai``` block, using the `item`, `data`, and `ai_output`, which is the string returned by the AI. 
        - It can save files in place or create new files. 
        - Later, it will even be able to queue new devai work.
- By default, this will run with `gpt-4o-mini` and look for the `OPENAI_API_KEY` environment variable.
- It supports all AI providers supported by the [genai crate](https://crates.io/crates/genai).
    - Here are the environment variable names per provider: `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `COHERE_API_KEY`, `GEMINI_API_KEY`, `GROQ_API_KEY`.
    - On Mac, if the environment variable is not present, it will attempt to prompt and get/save it from the keychain, under the devai group.

## Example of a Command Agent File

`.devai/defaults/proof-comments.md` (see [.devai/defaults/proof-comments.md`](./_base/agents/proof-comments.md))

## Config

On `devai run` or `devai init` a `.devai/config.toml` will be created with the following:

```toml
[genai]
# Required (any model rust genai crate support).
model = "gpt-4o-mini" 

[runtime]
# Default to 1 if absent. Great way to increase speed when remote AI services.
items_concurrency = 1 
```

## Future Plan

- Support for the `# Items` section with `yaml` or `Rhai`.
- More `Rhai` modules/functions.
- Support for `# Before All`, `# Before`, `# After`, and `# After All` (all `Rhai`).
- `--dry-req` will perform a dry run of the request by just saving the content of the request in a file.
- `--dry-res` will perform a real AI request but just capture the AI response in a file (the request will be captured as well).
- `--capture` will perform the normal run but capture the request and response in the request/response file.
