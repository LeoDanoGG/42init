*This project has been created for help in the development of the 42 curriculum by legomez.*

# 42 Project Initializer (`42init` for 42 Cursus)

<img src="https://skillicons.dev/icons?i=bash&theme=dark" width="150"/> 

An interactive CLI assistant (*wizard*) for 42 School students to quickly scaffold project workspaces, clone Vogsphere/GitHub repositories, generate language-specific boilerplate (C & Python), and keep your source files synchronized between the `Makefile` and the `README.md`.
 
## Table of contents
 
- [Key features](#key-features)
- [Requirements](#requirements)
- [Installation](#installation)
- [How to use](#how-to-use)
- [Syncing existing projects](#syncing-existing-projects)
- [Limitations](#limitations)
- [Important reminders](#important-reminders)
---
 
## Key features
 
1. **Interactive CLI wizard:** launch `42init` and answer simple prompts with safe defaults. Invalid answers are rejected and asked again.
2. **Two modes:**
   - **Create / Scaffold:** sets up a brand-new C or Python workspace.
   - **Sync / Update:** scans the current directory for `.c` files and updates both the `Makefile` and the `README.md`. A shortcut is available: `42init sync`.
3. **C and Python support:**
   - **C:** optional header file (`.h`) with a valid uppercase include guard, a `Makefile` with `-Wall -Wextra -Werror`, header dependency tracking, and optional linking against your own `libft`.
   - **Python:** an executable `main.py` entrypoint and an empty `requirements.txt`.
4. **Non-destructive:** files that already exist (`.h`, `Makefile`, `README.md`, `main.py`) are never overwritten; the script tells you which ones it skipped. It is safe to run on a cloned repository.
5. **Clean source grouping:** `.c` files are listed 5 per line with aligned continuation lines in the `Makefile` and inside a code block in the `README.md`.
6. **Tailored documentation:** the `README.md` template covers description, architecture, usage, testing, resources and AI usage disclosure.
7. **Automatic author:** the default author is your current login (`$USER`); you can type other logins for group projects.
---
 
## Requirements
 
- `bash`, `git`, and standard Unix tools (`awk`, `sed`, `find`)
- macOS or Linux (written and tested with portable syntax, no GNU-only flags)
---
 
## Installation
 
Clone the repository once and point a global alias at the script. Updating is then a simple `git pull`.
 
1. **Clone it:**
```bash
   mkdir -p ~/scripts
   git clone https://github.com/LeoDanoGG/42Madrid_init ~/scripts/42init
   chmod +x ~/scripts/42init/42init.sh
```
 
2. **Create the alias for your shell:**
   - **Zsh** (default shell on 42 Macs):
```bash
     echo "alias 42init='~/scripts/42init/42init.sh'" >> ~/.zshrc
     source ~/.zshrc
```
 
   - **Fish:**
```fish
     alias 42init='$HOME/scripts/42init/42init.sh'
     funcsave 42init
```
     `funcsave` persists the alias for all future sessions.
 
3. **Update later:**
```bash
   git -C ~/scripts/42init pull
```
 
> 💡 **Default author:** the script uses your current login (`$USER`) when you just press Enter at the author prompt. No editing needed.
 
---
 
## How to use
 
Run the command from the directory where you want the project folder to be created:
 
```bash
42init
```
 
### Interactive flow
 
1. **Action:**
   1. Create / Scaffold a new project
   2. Sync / Update `.c` files in the current directory *(Makefile & README)*
2. **Project name:** letters, digits, `.`, `_` and `-` only (no spaces or slashes).
3. **Repository setup:** local directory, or Git clone (`git@vogsphere...` / GitHub). The URL is required if you choose to clone.
4. **Author login(s):** credited in `README.md` (and in `main.py` for Python projects). Default: your `$USER`.
5. **Language:**
   1. C (Common Core / Piscine)
   2. Python (Common Core)
6. **C options** *(only if C is selected)*:
   - Generate `<name>.h`? `[Y/n]`
   - Generate a `Makefile`? `[Y/n]`
   - Link against your own `libft` (`./libft`)? `[y/N]`
7. **Documentation:** generate `README.md`? `[Y/n]`
If you cloned a repository that already contains `.c` files in its root, they are added to `SRCS` and to the README list automatically.
 
### About the header option
 
Say **no** to the header when a cloned repository already ships its own. The `Makefile` links the header as a dependency of every `.o` file through the `HEADER` variable, which is filled in only if `<name>.h` actually exists. If it does not (for example, your header has another name), `HEADER` is left empty and `make` keeps working; set it by hand to the right file so that editing it triggers a recompilation.
 
### About the `libft` option
 
The generated `Makefile` expects your library in `./libft`, with its own `Makefile` producing `libft.a`. Copy the **sources, its `Makefile` and `libft.h`** (not the `.o` / `.a` files, and without a nested `.git` folder).
 
- It adds `-I./libft`, so you can write `#include "libft.h"`.
- If `libft/libft.a` does not exist, `make` builds it with `make -C libft` before linking.
- `clean`, `fclean` and `re` are propagated to libft as well.
- A second `make` with nothing changed does nothing.
**If you edit libft while working on the project**, `make` will not notice, because `libft.a` already exists. Run `make -C libft` (or `make re`) to pick up the changes. In an evaluation this is not an issue, since the repository is cloned and built from scratch.
 
### Using a different library
 
The `libft` option is a **template**. If you need another library with the same layout (a folder with its own `Makefile` that builds a `.a`), generate the project with this option and then edit the generated `Makefile` by hand. A comment above the variables reminds you of where:
 
| What 			| Where 						| Example 						|
|---------------|-------------------------------|-------------------------------|
| Folder 		| `LIBFT_DIR` 					| `./mylib` 					|
| Archive name 	| `LIBFT` 						| `$(LIBFT_DIR)/libmylib.a` 	|
| Link flag 	| `-lft` in the `$(NAME)` rule 	| `-lmylib` 					|
 
The rules (`make -C`, `clean`, `fclean`) and include path (`-I`) follow `LIBFT_DIR`, so they need no changes.
 
Libraries that need more than that, such as MiniLibX (extra system flags, its own build process), are **not** covered by the template: check the subject and add the flags yourself.
 
---
 
## Syncing existing projects
 
Once you have added or removed `.c` files in your project root, you do not need to edit `SRCS` or the README list by hand:
 
```bash
cd my_project
42init sync
```
 
*(Or run `42init` and choose option 2.)*
 
The script scans for `.c` files, groups them 5 per line, and rewrites only what is **between the markers**:
 
| File 			| Start marker 				| End marker 			|
|---------------|---------------------------|-----------------------|
| `Makefile` 	| `# SRCS_START` 			| `# SRCS_END` 			|
| `README.md` 	| `<!-- FILES_START -->` 	| `<!-- FILES_END -->` 	|
 
If a file or its markers are missing, that file is skipped with a warning and nothing else is touched. Running sync repeatedly gives the same result.
 
> If you cloned a repository that already had its own `README.md`, the script will not overwrite it. To use sync with it, add the two README markers wherever you want the list to appear.
 
---
 
## Limitations
 
- Sync only scans the **project root** (no subdirectories such as `src/`).
- Sync only handles **`.c` files**. Python projects do not use markers.
- Every `.c` in the root is listed, including test files such as a throwaway `main.c`. Remove them before syncing if the subject does not allow them.
- Anything you write between the markers is replaced on the next sync. Keep your own notes outside of them.
---
 
## Important reminders
 
> ⚠️ **Preserve the markers.** Do not delete or rename `# SRCS_START` / `# SRCS_END` in the `Makefile` or `<!-- FILES_START -->` / `<!-- FILES_END -->` in the `README.md`.
 
> ⚠️ **Makefile tabs.** Recipe lines must start with a real TAB. If you edit the generated `Makefile` and get `missing separator`, your editor probably converted tabs into spaces.
 
> ⚠️ **42 header.** This script creates structural boilerplate only. Insert the official 42 header (`:Stdheader` in Vim / VS Code extension) at the top of your `.c`, `.h` and `Makefile` files, as your subject and evaluation require. Norminette itself checks `.c` and `.h` files; check your subject for the rest.
 
> ⚠️ **Check the subject.** The generated files are a starting point. Always verify the final `Makefile` and `README.md` against the requirements of your specific project.
