# Environment Variables

## modifying and accessing environment variables

The following commands are used to view, set, and unset environment variables:
- printenv : prints current environment variables
- env : can print current variables, or run a command in a new environment (sets and unsets variables in the new environment)
- export : sets an environment variable
- unset : removes the value of an environment variable

![Using printenv](resources/experiment_2/printenv.png)

![Using export and unset](resources/experiment_2/export.png)

## using curl

Curl can be used to transfer files from the host machine to a virtual machine. I used python's built in httpserver to host a folder, then used curl requests to grab all the files.

![Transfer using Curl](resources/experiment_2/curl.png)

## fork()

In this section, we will show that child processes inherit the environment variables of their parent.

We use the following program which forks itself, and will print either the child or parent's environment variables.

![Executable that will print environment variables of either the parent or the child](resources/experiment_2/myprintenv.png)

Instead of creating a command argument that decides which process prints its variables, I just created two seperate files for simplicity's sake.

![Creating a file that prints parent environment variables, and one that prints child environment variables](resources/experiment_2/forkCreation.png)

I used diff after redirecting the output for both executables into seperate files. The only difference was the file name because I ran two seperate executables.

![No difference between parent printenv and child printenv](resources/experiment_2/forkOutputDiff.png)

## execve()

Execve does not create a new process when executing a command. It will instead overwrite all the data of the current process. execve has the following function signature,
where argv refers to the command arguments to the new executable and envp refers to the environment variables to the environment of the new execution.

```c
int execve(const char *pathname, char *const argv[], char *const envp[]);
```

We will prove that envp determines what environment variables are set/unset in the execution environment.
First we use this snippet that calls env, which by default just prints the environment variables.

![Snippet of code that uses execve to run 'env'](resources/experiment_2/myenv.png)

Passing in NULL should print nothing (as no environment variables are passed).

![Running execve 'env' without passing environ](resources/experiment_2/task3_0.png)

Passing in environ should print all the environment variables of the current shell.

![Running execve 'env' without passing environ](resources/experiment_2/task3_1.png)

## system()

system uses execve to run a new shell and passes the program as an argument to the shell. System also passes environ by default.

![Snippet used to test if system('env') maintains environment variables](resources/experiment_2/task4_0.png)

The above screenshot proves this true, as execve env without passing environ prints nothing.

## setUID programs

setUID programs allow users to run programs that have escalated privilages. Linux programs inherit their parents UID, so system and execve are potential vectors of attack if we can cause them to open malicious programs.
We used the following snippet to print the system environment variables after using `export` to set MYVAR, PATH, and LD_LIBRARY_PATH to 'custom'.

![Snippet used to test if system('env') maintains environment variables](resources/experiment_2/task5_0.png)

Looking through the environment variables, I found that MYVAR and PATH were modified, but LD_LIBRARY_PATH was not.
LD_LIBRARY_PATH is protected to avoid linux from linking malicious code instead of system defaults. PATH is not protected, because kernel programs don't use PATH: its purely a user-space vulnerability.

Modern shells have protections implemented for setUID programs, and don't permit PATH modification for setUID. We use zsh to test, as it doesn't have this protection.

## Exploiting PATH with ZSH

