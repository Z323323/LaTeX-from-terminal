# LaTeX from terminal

This article contains the complete process required to generate (scientific) documentation with LaTeX directly from terminal (for FREE) **in an Ubuntu environment (22 LTS)**. It also contains some utilities which can be really helpful.

## Couple motivations

If you read the last part of [https://github.com/Z323323/Spacemacs-for-LaTeX], you'll find out the actual limits of that implementation, and also, that, in the end, Spacemacs is definitely useless since everything mentioned. So I started thinking about a pure terminal solution, along with some bash utilities to speed up the work. After maaany battles using Gemini I've found the final detail to solve the infamous ```pygmentize``` problem.

## Download texlive-full and Pygments

Follow [https://github.com/Z323323/Spacemacs-for-LaTeX?tab=readme-ov-file#install-tex-live] and [https://github.com/Z323323/Spacemacs-for-LaTeX?tab=readme-ov-file#another-couple-downloads-before-using-spacemacs].

## Setup your project directory

This is always the best way to start. You can find a clear example at [https://github.com/Z323323/Spacemacs-for-LaTeX?tab=readme-ov-file#pre-create-a-good-directory-structure-for-your-latex-project]. Now, instead of the ```.projectile``` file, you need to insert a ```.latexmkrc``` file. This works exactly as ```.bashrc``` or ```.zshrc``` but for the LaTeX engine, which is in the end the solution to the whole problem.

## Define a couple utilities to speed up some stuff

If you worked enough with LaTeX stuff from command line, you'll know everytime you compile your project, a lot of "auxiliary" files are generated, and are necessary to build the final document. Now, it's true that you need those files to generate the final ```pdf``` but it's also true that sometimes you need to clear up the whole directory and restart the build process (e.g. when you need to reformat index or bibliography, or both). I defined two utilities which could be useful in other contexts too, and called them ```generate_initial_project_tree_list.sh``` and ```restore_initial_project.sh```. Their functioning is very simple. The first one simply generates a white list of files and directories, and the second one clean all files and directories which are not part of the white list (you should use this one after you compiled your LaTeX project and want to rebuild from scratch).

WARNING: Make sure you run ```generate_initial_project_tree_list.sh``` in an isolated environment, that is, the folder where you run it must ONLY contain your project. Actually this one is not that dangerous, but, for example, if you run it under ```/```, it will generate a file called ```initial_project_tree_list.txt``` containing the contents of your whole system (at least I guess so).

ANOTHER IMPORTANT NOTE: Since the second utility is completely independent, you CAN (and it's even better if you do it), modify the contents of ```initial_project_tree_list.txt``` as you want, creating the whitelist you want. We can say ```generate_initial_project_tree_list.sh``` is really useful to speed up the process initially. Below its content.

**LAST (but not least) NOTE**: This script and every other script I developed (well that was mainly Gemini's work but, you know, it can't take credits so...) **are meant to be used once you positioned yourself in the project directory**. Please don't forget this thing. This means you just need to write ```cd /???/your-project``` before running executables. Also the script below should always be the first you run, otherwise it will be quite useless.

```bash
#!/bin/bash

# Finds every file and directory inside the project and saves them into a list.
# WARNING: This script is powerful, remember to use it into an isolated project.

PROJECT_PATH="$PWD"

find "$PROJECT_PATH" -mindepth 1 -type f -o -type d > "$PROJECT_PATH/initial_project_tree_list.txt"

echo "Complete list of all files and directories created in initial_project_tree_list.txt."
```

Now, the second one is the one which is really **dangerous**.

**WARNING**: If you run this script inside the wrong folder, say ```/``` **you will delete your whole system** (except your whitelist :'''D, this would be quite funny ngl). If you run this script with the wrong ```initial_project_tree_list.txt``` **you will delete your whole project** (this one actually happened to me because I run the previous one in the wrong folder and then moved the file into another project). As I already told you this script deletes all the files which are not contained into the whitelist file. Code below.

```bash
#!/bin/bash

PROJECT_PATH="$PWD"
LIST_FILE="initial_project_tree_list.txt"

# --- Initial security controls ---
if [[ ! -f "$LIST_FILE" ]]; then
    echo "Error: File '$LIST_FILE' not found."
    echo "Ensure you generated initial_project_tree_list.txt using generate_initial_project_tree_list.sh."
    exit 1
fi

# ====================================================================
#  Phase 1: Read the list
#  Put generated initial_project_tree_list.txt into an array (keep_files)
# ====================================================================

declare -a keep_files
while IFS= read -r line; do
    keep_files+=("$line")
done < "$LIST_FILE"

# ====================================================================
#  Phase 2: Compare and clean
#  Iterate every element of the current project, compare with initial_project_tree_list.txt and remove if it's not part of initial_project_tree_list.txt
# ====================================================================

# print0 put the '\0' delimiter at the end of each file or directory found, then read removes it
find "$PROJECT_PATH" -mindepth 1 -print0 | while IFS= read -r -d $'\0' item; do
    
    # Ignore if item is initial_project_tree_list.txt
    if [[ "$item" == "$PROJECT_PATH/$LIST_FILE" ]]; then
        continue
    fi
    
    # Control if item is whitelisted
    is_in_list=false
    for file_to_keep in "${keep_files[@]}"; do
        if [[ "$item" == "$file_to_keep" ]]; then
            is_in_list=true
            break
        fi
    done
    
    # If item is NOT whitelisted, clean it
    if [[ "$is_in_list" == false ]]; then
        echo "Cleaning: $item"
        rm -rf "$item"
    fi
done

echo ""
echo "Cleaning completed! 🎉"
```

Stay safe boys, ```Cleaning completed! 🎉``` can easily turn into ```System destroyed! 🎉```. By the way, you can store these two scripts wherever you want. In order to make a structured solution i put them two into ```~/???/my_utils/generics```. I suggest you to do the same. Ofc, everytime you see ```???``` you'll have to substitute your name.

## Define the actual solution

Well well, here we are. First things first: the solution is not hard, and this the good news, but it's better to include some context otherwise you won't understand this magics. Initially, the best thing to do is to edit the file ```.latexmkrc``` you created previously and put the content below inside.

```
$pdf_mode = 1;
$recorder = 1;

# Defines the absolute path to find pygmentize
$ENV{PATH} = "/home/???/.local/bin:$ENV{PATH}";

# Defines every path of PYTHONPATH to make pygmentize work
$ENV{PYTHONPATH} = "/usr/lib/python3.10:/usr/lib/python3.10/lib-dynload:/home/???/.local/lib/python3.10/site-packages:/usr/local/lib/python3.10/dist-packages:/usr/lib/python3/dist-packages:$ENV{PYTHONPATH}";

# Enables the execution of external scripts
$pdflatex = 'pdflatex -synctex=1 -interaction=nonstopmode -shell-escape --shell-escape';
```

Not sure if the last line is actually helpful but the solution works so just leave it there, it won't cause problems. The actual important details are the environment variables defined above. These look quite strange and it's because they are defined by the LaTeX engine using the Perl engine. These are the solution to every problem because in order to execute ```pygmentize``` LaTeX must find the executable, and in turn, ```pygmentize``` must find other dependencies (```pygments```). If you add the paths showed above to your system's environment variables, the LaTeX engine won't see them because for some arcane reason the LaTeX engine has its own (Perl based) environment variables and doesn't see any other environment variable. When the engine executes ```pygmentize``` through ```-shell-escape``` it won't see its path unless you specifically list it under its environment variables, and also, ```pygmentize``` won't see ```PYTHONPATH``` variables defined through your system because the script is executed in the "context" of the LaTeX engine and so you'll have to add ```PYTHONPATH``` too. The above solution should work for you too, but I suggest to check "```PYTHONPATH```" by running ```python3 -c "import sys; import pprint; pprint.pprint(sys.path)"```, and the correct directories manually.

Now that we have ```.latexmkrc``` correctly configured we just need to setup a bash script to run every single script needed to compile our final ```pdf```. In order to build as fast as possibile I created two different scripts: one to compile from scratch (with index and bibliography), and one to compile modifications to existing (.tex) files in the fastest way (once we have the index and bibliography correctly compiled we'll just need to modify .tex files and compile quickly). Below the code for the complete build process (```complete_build.sh```) and the code for the fast build process (```fast_build.sh```).

```bash
#!/bin/bash

PROJECT_PATH="$PWD"

# Compila il documento per la prima volta per generare i file ausiliari
echo "Build step (1/5), generating aux files..."
latexmk -pdf -synctex=1 -interaction=nonstopmode -shell-escape "$PROJECT_PATH/main.tex"

echo "Build step (2/5), generating bibliography with biber..."
biber "$PROJECT_PATH/main.bcf"

echo "Build step (3/5), generating index with makeindex..."
makeindex "$PROJECT_PATH/main.idx"

echo "Build step (4/5), recompiling project with bibliography and index..."
latexmk -pdf -synctex=1 -interaction=nonstopmode -shell-escape "$PROJECT_PATH/main.tex"

echo "Build step (5/5), compiling one last time to resolve references..."
latexmk -pdf -synctex=1 -interaction=nonstopmode -shell-escape "$PROJECT_PATH/main.tex"

echo "Complete build completed! 🎉"
```

```bash
#!/bin/bash

PROJECT_PATH="$PWD"

echo "Build step (5/5), compiling one last time to resolve references..."
latexmk -pdf -synctex=1 -interaction=nonstopmode -shell-escape "$PROJECT_PATH/main.tex"

echo "Complete build completed! 🎉"
```





