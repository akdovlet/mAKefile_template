# mAKefile_template
```NAME	:= genius_name``` &#8592; Variable containing the name of your program or library

```LIBDIR	:=	lib``` &#8592; Folder containing your external libraries (libft, mlx or whatever)

```LIBFT	:= 	$(LIBDIR)/libft/libft.a``` &#8592; Path to your external library archive file (ex:```libft.a```). Create one for each library.

```
SRC		:=	genius_code.c	\
			seed_phrase.c
```
&#8593; Explicitly declare your source files. GNU Make uses a line-based syntax, where newline indicates the end of a statement. However, for readability, we can split the assignment of SRC over multiple lines by escaping the newline character with a backslash (this is only true in non-recipe statements).

```SRC_DIR	:=	src``` &#8592; Name of your folder containing your source files

```BUILD	:=	.build``` &#8592; Name of your folder where your build files will be stored, such as .o and .d files.

```SRC 	:=	$(addprefix $(SRC_DIR)/, $(SRC))``` &#8592; addprefix is a function that takes 2 arguments, (1) a prefix and (2) a variable receiving said prefix. This will simply add the path to src files, and will be as such: ```SRC := src/genius_code.c src/seed_phrase.c```

```OBJ 	:=	$(patsubst $(SRC_DIR)/%.c, $(BUILD)/%.o, $(SRC))``` &#8592; patsubst (pattern substitute), GNU Make builtin function that, like the name implies, will match a pattern and replace it with another. Takes 3 arguments: (1) pattern to find, (2) if the pattern is found, replace it with the second argument; (3) last argument is where it will look for patterns. In this instance, we are searching through all our source files and doing two things:
+ Replacing .c suffix to .o to change the file type from source to object
+ Replacing the ```$(SRC_DIR)``` with ```$(BUILD)``` to change the directory of the object files from the source. So that they are not generated in the same folder.

Another common way of assigning your OBJ variable is ```OBJ := $(SRC:.c=.o)```, using another of GNU Make pattern substitution function, except this one only does simple suffix replacement. It would achieve the first part of patsubst but would not be able to do more complex substitutions. This would result in ```OBJ := src/genius_code.o src/seed_phrase.o```.

```DEPS 	:=	$(OBJ:.o=.d)``` &#8592; We will also be generating .d files. In GNU Make, they are dependency files generated by the compiler and contain a list of header files that the corresponding source files depend on. The -MMD and -MP flags are used for generating them. This is how we will create dependency between .c files and their .h files. If a header is modified, every source file using said header will be recompiled.

```CC		:=	cc``` &#8592; GNU Make uses predefined variables, which you can look up with ```make -p```. For example, we never explicitly defined the variable ```$(AR)``` but if we were to call $(AR) in a recipe it would expand to whichever archiving program your system uses; for me it would be 'ar'. ```$CC``` is also implicitly defined by default and expands to 'cc'. So this assignment is not necessary but is here in case someone doubts you are compiling with cc. (You can always use ``` make -p | grep 'CC :='``` to prove your point).

```CFLAGS	:=	-Wall -Werror -Wextra -MMD -MP -Iinclude -I$(LIBDIR)/libft/include``` &#8592; Another implicit variable, except it doesn't contain anything by default; it's up to us to pass extra flags to the compiler. Fun fact: this variable is always called when compiling, even if you don't call it explicitly. Unless you change the compilation rule (```.o: .c```).

```-MMD``` Will generate a list of dependency (headers) for each source file. ```-MP``` is used to prevent Make from generating an error and failing if a header file has been moved, renamed or deleted. Make will still be able to run, and eventually the compilation will fail, giving you a more meaningful error message from the compiler. It's a way to make the build process more robust to project changes.

```-I$(LIBDIR)/libft/include``` is used to tell the compiler where to look for headers you included. Useful when your header files are in a different folder than your source files.

```all: create_dirs $(NAME)``` &#8592; 'all' is the first rule and the one that will be called when we do ```make```. It will generate all the high level targets. A rule is written as such: 
```
rule_name: target1 target2 ...
	recipe(commands) to create or update target
```
from GNU Make's manual: "A rule appears in the makefile and says when and how to remake certain files." The 'how' is the recipe, but the 'when' is less obvious. Make will run the rule if the target either doesn't exist or if its access time is more recent than the last run. In this case, 'all' will call another rule 'create_dirs' and then call the rule ```$(NAME)```.

```create_dirs: $(BUILD)``` &#8592; 'create_dirs' will not call $(BUILD) if the folder already exists. This is to prevent running ```mkdir``` unnecessarily.

```make
$(BUILD):
	@if [ ! -d "$(BUILD)" ]; then mkdir $(BUILD); fi
```
&#8593; This recipe could just be ```mkdir $(BUILD)``` but it's a nice display of how you can do conditional statements in a Makefile. ```-d``` is for directory, so if .build dir doesn't exist then do ```mkdir .build```

```make
$(NAME): $(OBJ) $(LIBFT)
	@$(CC) $(CFLAGS) $(OBJ) $(LIBFT) -o $(NAME)
```
&#8593; This is the linking rule. It will create our executable from our .o files and libft.a file.

```make
$(BUILD)/%.o: $(SRC_DIR)/%.c
	@$(CC) $(CFLAGS) -c $< -o $@
	@printf "\033[1;32%sm\tCompiled: $<\033[0m\n";
```
&#8593; This is a pattern rule that tells make how to build .o files. Even though we never call this rule explicitly, Make will use it to build our .o files, and for each file it will check with its corresponding .c file. On subsequent runs, a .o will be recreated only if its associated .c file is more recent.

```make
$(LIBFT):
	@$(MAKE) --no-print-directory -C  $(LIBDIR)/libft
```
&#8593; Simple rule to run ```make``` in another directory using the ```-C``` flag. ```--no-print-directory``` is for aesthetics sake, so that ```make``` does not print which directory it enters and leaves.

```make
clean:
	@if [ -d "$(BUILD)" ]; then $(RM) -rf $(BUILD) && echo "\033[1;31m\tDeleted: $(NAME) $(BUILD)\033[0m"; fi
	@$(MAKE) --no-print-directory clean -C $(LIBDIR)/libft
```
&#8593; More conditional statements

```make
fclean: clean
	@if [ -f "$(NAME)" ]; then $(RM) -rf $(NAME) && echo "\033[1;31m\tDeleted: $(NAME)\033[0m"; fi
	@$(MAKE) --no-print-directory fclean -C $(LIBDIR)/libft
```
&#8593; ```-f``` for file.

``` make
re: fclean all
```

```make
-include $(DEPS)
```
&#8593; This directive tells Make to ignore any errors if .d files don't exist. Useful when you first run ```make``` when no .d files have been generated yet.

```make
.PHONY: all create_dirs clean fclean re
```
&#8593; You can try this at home: create a file called whatever.c with some random code. Don't add it to your 'src' variable in your makefile. Now do ```make whatever```; what happens? Yeah, well that's what ```.PHONY``` is for. It prevents compiling files that share the same name as one of your rules. 

Makefile, unlike C, does not run from top to bottom, it will build according to dependencies. As shown below:

![Untitled-2024-05-17-0233](https://github.com/akdovlet/mAKefile_template/assets/86743971/2502ec20-48e3-4481-926d-3865d7bad597)


Extra tips:

```make -j```: Will allow Make to use all of your cpu's threads. Very fast compilation.

```make -n```: A dry run of ```make```, will show you exactly what the next ```make``` call will do. Useful for knowing if you makefile relinks or not.

```make -k```: Tells Make to go as far as possible, usually Make stops at the first error.

```make -s```: run ```make``` in silent mode, does not print the commands as they are executed (You can do this more precisely by adding ```@``` in front of a command.

```make -B```: will consider all targets as out of date and execute every rule necessary to update them.

Will update my template as I get more knowledge. Peace.
