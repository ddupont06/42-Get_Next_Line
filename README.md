<h1 align="center">
	Get Next Line
</h1>

<p align="center">
	<img src="https://user-images.githubusercontent.com/91064070/147373286-a6039361-2208-431a-9625-384bf44a64d9.png?raw=true" alt="Sublime's custom image"/>
</p>

## Result

/![](result.png)

## How It Works

In the following example, we have a file indexed by a [file descriptor](https://en.wikipedia.org/wiki/File_descriptor) (`fd`) that contains 2 lines. We will set the buffer size (`BUFFER_SIZE`) to 9 and incrementally use the `read()` function to [read](https://man7.org/linux/man-pages/man2/read.2.html) up to 9 bytes from the `fd`. In addition, we will use a [static variable](https://en.wikipedia.org/wiki/Static_variable) (`static`) as a read buffer to access what was read in the previous calls.

The content of the file is:

```
First line
Second line
```

Equivalent to:

```c
 1|2|3|4|5|6|7|8|9|   |1| 2|3|4|5|6|7|8|9|   |1|2|3|4| 5 |
[F|i|r|s|t| |l|i|n|] [|e|\n|S|e|c|o|n|d| |] [|l|i|n|e|EOF|]
```

After the first call of `read()`, `static` is:

```
static = [First lin]
```

No newline (`\n`) or end-of-file (`EOF`) is found, so we call again `read()` and concatenate the previous string contained in `static` with the next 9 characters readed from the `fd`.

```
static = [First line\nSecond ]
```

Here, `\n` is found, so we extract from `static` the part of the string up to and including `\n`, null-terminate it, and return it.

```
returned line = [First line\n]
```

Then, we update `static` for the next call by keeping the remaining characters that were not returned.

```
static = [Second ]
```

If the function is called again, the process is repeated until `\n` or `EOF` is reached.

```
static = [Second lineEOF]
returned line = [Second lineEOF]
```

Note that to allow the program to handle mutiple `fd` at the same time (cf. bonus part in [`subject`](Get_Next_Line.pdf)), `static` is declared in [`get_next_line_bonus`](get_next_line_bonus.c) as an array of strings where each row is corresponding to one file descriptor.

## Usage

Clone this repository and access its folder:

```shell
git clone https://github.com/ddupont06/42-Get_Next_Line.git && cd 42-Get_Next_Line
```

Create a main file:

```shell
touch main.c
```

Below is an example of main file. You can replace `read_file()` by `read_line()` in the `main()` function to display only one line instead of all the lines per `fd`. 

```C
#include "get_next_line.h"
#include <fcntl.h>
#include <stdio.h>

static void	read_line(int fd)
{
	char	*line;

	printf("\nfd = %d:\n", fd);
	if ((line = get_next_line(fd)))
	{
		printf("%s", line);
		free(line);
	}
}

static void	read_file(int fd)
{
	char	*line;

	printf("\nfd = %d:\n", fd);
	while ((line = get_next_line(fd)))
	{
		printf("%s", line);
		free(line);
	}
}

int	main(int ac, char **av)
{
	int	fd[ac];
	int	i;

	fd[0] = open(av[1], O_RDONLY);
	if (!av[1])
	{	
		close(fd[0]);
		fd[0] = 0;
	}
	read_file(fd[0]);
	i = 1;
	while (av[++i])
	{
		fd[i - 1] = open(av[i], O_RDONLY);
		read_file(fd[i - 1]);
	}
	while (--i > 0)
	{
		printf("\nClosing fd[%i]\n", fd[i - 1]);
		close (fd[i - 1]);
	}
}
```

Compile all the files while choosing a `BUFFER_SIZE`:

```shell
gcc -D BUFFER_SIZE=<size_to_choose> get_next_line_bonus.c get_next_line_bonus.h get_next_line_utils_bonus.c main.c -o get_next_line
```

Execute your program with as many files as you want to read as arguments:

```shell
./get_next_line <name_of_first_file> <...> <name_of_last_file>
```

Since the integer `fd` associated with the standard streams [stdin](https://en.wikipedia.org/wiki/Standard_streams#Standard_input_(stdin)), [stdout](https://en.wikipedia.org/wiki/Standard_streams#Standard_output_(stdout)), and [stderr](https://en.wikipedia.org/wiki/Standard_streams#Standard_error_(stderr)) are 0, 1, and 2, respectively, the first integer `fd` of the outputs of the function will begin at 3. Moreover, stdin is supported when there is no input arguments and on errors a `fd` of -1 is returned.
