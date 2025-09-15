# LaTeX from terminal

This article contains the complete process required to generate (scientific) documentation with LaTeX directly from terminal (for FREE) **in an Ubuntu environment (22 LTS)**. It also contains some utilities which can be really helpful.

## Couple motivations

If you read the last part of [https://github.com/Z323323/Spacemacs-for-LaTeX], you'll find out the actual limits of that implementation, and also, that, in the end, Spacemacs is definitely useless since everything mentioned. So I started thinking about a pure terminal solution, along with some bash utilities to speed up the work. After maaany battles using Gemini I've found the final detail to solve the infamous ```pygmentize``` problem.

## Setup your project directory

This is always the best way to start. You can find a clear example at [https://github.com/Z323323/Spacemacs-for-LaTeX?tab=readme-ov-file#pre-create-a-good-directory-structure-for-your-latex-project]. Now, instead of the ```.projectile``` file, you need to insert a ```.latexmkrc``` file. This works exactly as ```.bashrc``` or ```.zshrc``` but with the LaTeX engine, which is in the end the solution to the whole problem.

## Define a couple utilities to speed up some stuff

If you worked enough with LaTeX stuff from command line, you'll know everytime you compile your project, a lot of "auxiliary" files are generated, and are necessary to build the final document. Now, it's true that you need those files to generate the final ```pdf``` but it's also true that sometimes you need to clear up the whole directory and restart the build process (e.g. when you need to reformat index or bibliography, or both). I defined two utilities which could be useful in other contexts too, and called them ```generate_initial_project_tree_list.sh``` and ```restore_initial_project.sh```. Their functioning is very simple. The first one simply generate a white list of files and directories, and the second one clean all files and directories which are not part of the white list (you should use this one after you compiled you LaTeX project and want to rebuild from scratch).

WARNING: Make sure you run ```generate_initial_project_tree_list.sh``` in an isolated environment, that is, the folder where you run it must ONLY contain your project. Actually this one is not that dangerous, but, for example, if you run it under ```/```, it will generate a file called ```initial_project_tree_list.txt``` containing the contents of your whole system (at least I guess so).

ANOTHER IMPORTANT NOTE: Since the second utility is completely independent, you CAN (and it's even better if you do it), modify the contents of ```initial_project_tree_list.txt``` as you want, creating the whitelist you want. We can say ```generate_initial_project_tree_list.sh``` is really useful to speed up the process initially. Below its content.

```bash
#!/bin/bash

# Find every file and directory inside the project and saves them into a list.
# WARNING: This script is powerful, remember to use it into an isolated project.

PROJECT_PATH="$PWD"

find "$PROJECT_PATH" -mindepth 1 -type f -o -type d > "$PROJECT_PATH/initial_project_tree_list.txt"

echo "Complete list of all files and directories created in initial_project_tree_list.txt."
```
