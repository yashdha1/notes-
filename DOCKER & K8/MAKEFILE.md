Run and compile your programs more efficiently with this handy automation tool. Are used the projects where a setup of things are needed In many programming languages (especially C/C++), building a program involves multiple steps:
- Compiling source files (`.c`, `.cpp`) into object files (`.o`).
- Linking object files into an executable.
- Rebuilding only files that changed.
MakeFiles makes some use of some simple rules for the project to be working in the production enviorment. 
eg. 
format : 

target: dependencies
command
     **target** → the file to create (e.g., `app`)
	 **dependencies** → files the target depends on (e.g., `main.o`)
     **command** → shell command to build the target

Example in the workings. 
app: main.o utils.o
gcc main.o utils.o -o app
	 To build `app`, you need `main.o` and `utils.o`
	 If either changes, rebuild `app`

Why Makefile tho? 
1. Useful in Automation workflows. 
2. Incremental Builds (Efficiency) -> rebuild shit thats changed only, No need to rebuild all the files. 
3. Understands the intelligent relationships. 
4. Standard in C / C++ projects. 


This shit is useless in Containarization tech and CICD workflows. 