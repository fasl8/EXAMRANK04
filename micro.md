# print error messages:
```
void	ft_putstr_fd2(char *str)
{
	int	i;

	i = 0;
	while (str[i])
		i++;
	write(2, str, i);
}
```
# executing commands:
```
int ft_execute(char **argv, int i, int tmp_fd, char **env)
{
```
1. **argv: command and its arguments.
2. i: index where the command ends in the argv array.
3. tmp_fd: file descriptor used for input redirection.
4. **env: environment variables.
5. ``` argv[i] = NULL; ``` sets the element in the argv array at index i to NULL.
6. ``` close(tmp_fd); ``` closes the file descriptor tmp_fd
7. ``` execve(argv[0], argv, env); ``` execute the command specified in argv[0], It replaces the current process image with a new process image as specified by the given command
```
    ft_putstr_fd2("error: cannot execute ");
    ft_putstr_fd2(argv[0]);
    write(2, "\n", 1);
```
8. If execve fails->prints an error message to the standard error output (file descriptor 2)
9. ``` return (1); ``` returns 1 to indicate that there was an error during command execution. 

# main
```
int main(int argc, char **argv, char **env)
{
```
1. argc: the number of command-line arguments.
2. **argv: command-line arguments.
3. **env: array of strings containing the environment variables.
```
    int i;
    int pid;
    int fd[2];
    int tmp_fd;
    (void)argc;

    pid = 0;
    i = 0;
    tmp_fd = dup(STDIN_FILENO);
```
4. pid (process ID)
5. fd :to store file descriptors for a pipe
6. tmp_fd :temporary file descriptor for input redirection -> assigned the result of duplicating the standard input file descriptor STDIN_FILENO using dup.
7. # iterates through the command-line arguments:
```
    while (argv[i] && argv[i + 1])
    {
```
8. ``` argv = &argv[i + 1]; ``` argv is adjusted to point to the next command in the arguments list
9. ``` i = 0; ``` reset to 0 to iterate over the new command's
10.  increments i until it reaches the end of the current command's arguments or encounters a semicolon (;) or pipe (|).
```
        while (argv[i] && strcmp(argv[i], ";") && strcmp(argv[i], "|"))
            i++;
```
12.  # checks if the command is cd:
```
	if (strcmp(argv[0], "cd") == 0)
	{
```
13.  checks the number of arguments. If it's not 2, it prints an error message
```
if (i != 2)
                ft_putstr_fd2("error: cd: bad arguments\n");
```
14.  attempts to change the directory using chdir and prints an error message if it fails.
```
            else if (chdir(argv[1]) != 0)
            {
                ft_putstr_fd2("error: cd: cannot change directory to ");
                ft_putstr_fd2(argv[1]);
                write(2, "\n", 1);
            }
        }
```
15.  # handling commands separated by semicolons (;):
```
else if (argv != &argv[i] && (argv[i] == NULL || strcmp(argv[i], ";") == 0))
{
```
16. checks if the current command ends with a semicolon (;). It also ensures that the command is not empty and that there is at least one more command after the semicolon.
17. ``` pid = fork(); ``` forks a child process.
# child process:
18. sets up the input redirection using dup2 to make the temporary file descriptor tmp_fd a duplicate of the standard input file descriptor STDIN_FILENO, execute the current command. If execution fails, it returns 1, indicating an error.
```
if (pid == 0)
{
    dup2(tmp_fd, STDIN_FILENO);
    if (ft_execute(argv, i , tmp_fd, env))
        return (1);
}
```
# parent process:
19. loses the temporary file descriptor tmp_fd,waits for the child process to finish using waitpid, duplicates the standard input file descriptor STDIN_FILENO to tmp_fd, preparing for the next command to be executed.
```
else
{
    close(tmp_fd);
    waitpid(-1, NULL, WUNTRACED);
    tmp_fd = dup(STDIN_FILENO);
}
```
20. # handles commands separated by pipes:
21. checks if the current command ends with a pipe (|), that the command is not empty.
```
else if (argv != &argv[i] && strcmp(argv[i], "|") == 0)
{
```
22. ``` pipe(fd); ``` creates a pipe using the pipe(), allowing the output of one process to be connected to the input of another.
23. ``` pid = fork(); ``` forks a child process.
# child process:
24. sets up the input redirection using dup2 to make the temporary file descriptor tmp_fd a duplicate of the standard input file descriptor STDIN_FILENO. sets up the output redirection using dup2 to make the write end of the pipe fd[1] a duplicate of the standard output file descriptor STDOUT_FILENO. closes the read end of the pipe (fd[0]) and the write end of the pipe (fd[1]). ft_execute to execute the current command. If execution fails, it returns 1, indicating an error.
```
if (pid == 0)
{
    dup2(tmp_fd, STDIN_FILENO);
    dup2(fd[1], STDOUT_FILENO);
    close(fd[0]);
    close(fd[1]);
    if (ft_execute(argv, i , tmp_fd, env))
        return (1);
}
```
# parent process:
25. closes the write end of the pipe (fd[1]), closes the temporary file descriptor tmp_fd , waits for the child process to finish using waitpid. reads from the read end of the pipe (fd[0]) and duplicates it to tmp_fd for the next command to be executed. closes the read end of the pipe (fd[0]).
```
else
{
    close(fd[1]);
    close(tmp_fd);
    waitpid(-1, NULL, WUNTRACED);
    tmp_fd = dup(fd[0]);
    close(fd[0]);
}
```
26.  closes the temporary file descriptor tmp_fd and returns 0
```
    close(tmp_fd);
    return (0);
}
```

