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
1. argc :the number of command-line arguments
2. **argv :command-line arguments
3. **env (an array of strings containing the environment variables).
