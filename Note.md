# Build Your Own Text Editor

[toc]

## Chapter 1 设置

### 如何安装C compiler

**Windows环境**

我们需要在Windows下安装**Linux环境**，因为我们的文本编辑器需要使用`<termois.h>`头文件，Win下不可使用。推荐有两款：分别是 [Bash on Windows](https://msdn.microsoft.com/en-us/commandline/wsl/about) 或者 [Cygwin](https://www.cygwin.com/). 

> **Bash on Windows**：只在Win10-64位上工作。安装后用`bush`进入Linux环境。在`bash`下，运行`sudo apt-get install gcc make`来安装GNU Compiler Collection然后`make`程序。如果`sudo`花费了很长时间去做任何事情，你可能需要 [fix your /etc/hosts file](https://superuser.com/questions/1108197) 。
>
> **Cygwin**：当安装包让你选择安装的包时，在`devel`类下选择`gcc-core`和`make`程序。使用Cygwin，你需要执行 Cygwin terminal程序。Cygwin的地址和Window home地址是分开的。如果你安装在`C:\cygwin64`，那么你的 目录就在` C:\cygwin64\home\yourname `.如果你想在Cygwin之外使用编辑器编写代码，就可以在其中保存代码。

### `main()`函数

创建新的`kilo.c`文件并加上`main()`函数。（`kilo`是我们的项目名称）

```c
//kilo.c
int main() {
	return 0;
}
```

在`main()`中`return`后，程序退出，并把一个整数返回给操作系统。`0`表示成功。

我们需要先通过C compiler把程序编程可执行文件。我们可以像命令行一样操作它。

为了编译`kilo.c`，在shell里面输入`cc kilo.c -o kilo`.如果不出错，将会产生可执行文件`kilo`.`-o`作为输出。指定输出可执行被称作`kilo`.

为了运行`kilo`，输入`./kilo`然后按下**Enter**。程序不输出任何东西，但是你可以检查它的退出status（`main()`的返回值）。通过输入`echo $?`，然后应该输出0.

### 使用`make`编译

每次输入`cc kilo.c -o kilo`重新编译太过麻烦。`make`程序可以帮你编译程序。你只需要去提供`Makefile`去告诉他如何编译你的程序。

写一个新的`Makefile`，里面输入如下内容：

```makefile
kilo: kilo.c
	$(CC) kilo.c -o kilo -Wall -Wextra -pedantic -std=c99
```

第一行说明`kilo`是我们想建立的东西，`kilo.c`是需要被建立的东西。第二行指明了为了从`kilo.c`建立`kilo`所需要的指令。确保第二行是制表符而不是空格。你可以根据需要缩进C文件，但是**`Makefile`必须使用制表符（Tab）**。

我们加入了如下的指令：

* `$(CC)`是`make`默认扩展到`cc`的变量。
* `-Wall`代表“**all W**aring”.会使得编译器警告你非技术上的错误，但是被认为是坏的或者C语言可疑的用法。例如不初始化使用变量。
* `-Wextra`和`-pedantic`开启更多的警告.在这个教程中，不应该出现除了"unused variable"以外的其他警告。如果得到了其他警报，请检查你的代码和那一步的代码是一样的。
* `-std=c99`指定了C语言的版本标准（**st**an**d**ard）。这里指定的是 [C99](https://en.wikipedia.org/wiki/C99) 。C99允许我们在函数任何地方声明变量。而 [ANSI C](https://en.wikipedia.org/wiki/ANSI_C) 要求所有变量都在程序或者块的顶端定义。

现在我们有了`Makefile`文件，尝试使用`make`编译文件。

可能会输出make: `kilo\' is up to date. 它可以根据每个文件最后编辑的时间戳辨别最新版本的 kilo.c。只有编辑过的才会被重编译。在大型的项目中不会花费过多时间重新编译。

下一章我们会使终端进入*raw mode*，并且从用户读入单个按键。

## Chapter 2 输入raw mode

让我们尝试读入用户键盘输入：

```c
→ #include <unistd.h>
int main() {
→ char c;
→ while (read(STDIN_FILENO, &c, 1) == 1);
  return 0;
}
```

`read()`和`STDIN_FILENO`来自`<unistd.h>`。我们让`read()`从标准输入读入`1`字节放入变量`c`。一直这么做直到没有更多字节去读入。`read()`返回读入的字节，当读到末尾的时候会返回`0`。

当运行`./kilo`的时候，终端会链接到标准输入，这样你的键盘输入就会被读入到`c`变量。但是默认情况下你的终端将以**canonical mode**启动（也称为**cooked mode**）。在这个模式下，只有当用户输入**Enter**时才会把键盘输入输入到系统。这对许多程序很有用：这让用户输入一行文字，可以使用**Backspace**去修正错误。但是这不适合更复杂的用户界面，例如文本编辑器。我们希望当每一个键盘输入进来时，我们可以立刻做出响应。

我们想要的是**raw mode**.然而没有简单的开关可以把终端设置为**raw mode**。**raw mode**是通过关闭许多flag实现的，本章将进行逐行操作。

退出上述程序，使用`Ctrl-D`告诉`read()`读到了末尾。或者你可以使用`Ctrl-C`指示该过程立刻终止。

### 按下`q`终止

为了展示canonical mode是怎么工作的，我们让程序读到`q`的时候终止.

```c
#include <unistd.h>
int main() {
  char c;
  while (read(STDIN_FILENO, &c, 1) == 1 && c != 'q');
  return 0;
}
```

为了退出程序，你需要输入一行包含`q`的文字，然后按下回车。程序会一个一个字符读入，直到读到`q`。q之后的所有字符都不会被读入。`q`之后的字符在程序退出后会被放入shell里面。

### 关闭回显

我们可以设置属性通过：（1）使用`txgetattr()`将当前属性读入结构体。（2）自己编写结构体。（3）将编写后的结构体通过`tcsetattr()`去写新的终端属性。让我们以这种方式关闭ECHO功能。

```c
#include <termios.h>
#include <unistd.h>
void enableRawMode() {
  struct termios raw;
  tcgetattr(STDIN_FILENO, &raw);
  raw.c_lflag &= ~(ECHO);
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}
int main() {
  enableRawMode();
  char c;
  while (read(STDIN_FILENO, &c, 1) == 1 && c != 'q');
  return 0;
}
```

 `struct termios`, `tcgetattr()`, `tcsetattr()`, `ECHO`, 和 `TCSAFLUSH`都来自于` <termios.h> `。

 `ECHO`功能使你键入的每一个键都可以打印到终端上，所以你可以看见你在输入什么。这在canonical mode很有用，但是在raw mode下会出现麻烦。所以我们关闭它。该程序与上一步功能相同，就是不打印你输出的内容。（和终端里键入密码类似）。

程序退出后，依然不能打印出来字符。这个时候`Ctrl-C`开始一个新的一行，输入`reset`后按下**Enter**。 在大多数情况下，这会将您的终端重置为正常状态。失败的话，您总是可以重新启动终端。下一步，我们将解决这个问题。 

终端属性可以通过`tcgetattr()`读入到`termios`结构体。在修改后，通过`tcsetattr()`来应用。`TCSAFLUSH`指定何时应用这些更改：在这种情况下， 它等待所有待处理的输出写入终端，并丢弃所有尚未读取的输入。 

`c_lflag`作为local flag。其他还有`c_iflag`(input flag),`c_oflag`(output flag),`c_cflag`（control flag）。所有这些我们都必须进行修改以启用原始模式。

`ECHO`是一个位段（bitflag），定义为二进制` 00000000000000000000000000001000 `,用NOT操作符`~`产生

` 11111111111111111111111111110111 `，然后再进行AND运算。这一位会变成`0`。这样的翻转在C语言中很常见。

### 退出时禁用raw mode

让我们对用户友好一些，当我们退出的时候重新装回原来的终端属性。我们将会复制一份`termios`结构体，用`tcsetattr()`在退出时应用。

```c
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>
struct termios orig_termios;
void disableRawMode() {
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &orig_termios);
}
void enableRawMode() {
  tcgetattr(STDIN_FILENO, &orig_termios);
  atexit(disableRawMode);
  struct termios raw = orig_termios;
  raw.c_lflag &= ~(ECHO);
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}
int main() { … }
```

`atexit()`来自`<stdlib.h>` .我们使用它来注册我们的`disableRawMode()`函数，以便在程序退出时自动调用，无论它是通过main（）返回还是通过调用exit（）函数退出。 

我们储存了原本的终端属性` orig_termios `。

程序退出后，剩余的输入不再输入到shell中。这是因为程序退出时将`TCSAFLUSH`选项传递到`tcsetattr()`。在将更改应用到终端之前，它会丢弃所有未读的输入。 

### 关闭 canonical mode

还有一个flag`ICANON`允许我们关闭canonical mode。这说明我们将会一位一位读入，而不是一行一行读入。

```c
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>
struct termios orig_termios;
void disableRawMode() { … }
void enableRawMode() {
  tcgetattr(STDIN_FILENO, &orig_termios);
  atexit(disableRawMode);
  struct termios raw = orig_termios;
  raw.c_lflag &= ~(ECHO | ICANON);
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}
int main() { … }
```

`ICANON`来自`<termios.h>`.Input flag(在`c_iflag`区域)一般都是`I`开头。但是`ICANON`不是input flag，而是一个local flag。

这时输入`q`就会立刻退出。

### 展示键盘输入

为了更好地知道raw mode是怎么工作的，我们要打印出我们`read()`的字符。我们会打印出每一个字符的ASCII码值，以及其代表的字符（如果是可打印的）。

```c
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>
struct termios orig_termios;
void disableRawMode() { … }
void enableRawMode() { … }
int main() {
  enableRawMode();
  char c;
  while (read(STDIN_FILENO, &c, 1) == 1 && c != 'q') {
    if (iscntrl(c)) {
      printf("%d\n", c);
    } else {
      printf("%d ('%c')\n", c, c);
    }
  }
  return 0;
}
```

 `iscntrl()` 来自 `<ctype.h>`,  `printf()` 来自`<stdio.h>`.

 `iscntrl()`判断字符是否是控制字符。ASCII码0-31,127是控制字符，32-126是可打印字符。

一些有趣的事情：

* 方向键，page up, page down, home,end键会打印3或4位：`27`，`[`和一个或两个其他字符。这是*转义序列*，所有转移字符由`27`位开始，按下`Escape`发送单独的`27`作为输入。
* `Backspace`是`127`,`Delete`是一个4位转义字符。
* `Enter`是`10`,也是`\n`.
* `Ctrl-A`是`1`,`Ctrl-B`是`2`,Ctrl和A-Z的组合键为1-26。

### 关闭`Ctrl-C`和`Ctrl-Z`标志

`Ctrl-C`给当前进程传送`SIGINT`标志使其终止，`Ctrl-Z`给当前进程传送`STGTSTP`使其暂停。

```c
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>
struct termios orig_termios;
void disableRawMode() { … }
void enableRawMode() {
  tcgetattr(STDIN_FILENO, &orig_termios);
  atexit(disableRawMode);
  struct termios raw = orig_termios;
  raw.c_lflag &= ~(ECHO | ICANON | ISIG);
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}
int main() { … }
```

`ISIG`来自`<termios.h>`，跟`ICANON`一样，它是`I`开头但是不是input flag。

现在`Ctrl-C`可以被读作`3`,`Ctrl-Z`可以被读作`26`.

### 禁用`Ctrl-S`和`Ctrl-Q`

默认情况下，`Ctrl-S`和`Ctrl-Q`用作软件流控制。`Ctrl-S` 停止数据传输到终端，直到按`Ctrl-Q` 

```c
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>
struct termios orig_termios;
void disableRawMode() { … }
void enableRawMode() {
  tcgetattr(STDIN_FILENO, &orig_termios);
  atexit(disableRawMode);
  struct termios raw = orig_termios;
  raw.c_iflag &= ~(IXON);
  raw.c_lflag &= ~(ECHO | ICANON | ISIG);
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}
int main() { … }
```

`IXON`来自`<termios.h>`。`I`代表input flag.`XON` 来自`Ctrl-S`和`Ctrl-Q`产生的两个控制字符的名称.`XOFF`暂停传输，`XON`恢复传输。 

现在`Ctrl-S`可以被读作`19`,`Ctrl-Q`可以被读作`17`.

### 禁用`Ctrl-V`

在一些系统，当你输入`Ctrl-V`， 终端等待您键入另一个字符，然后按字面意义发送该字符。 例如，在禁用`Ctrl-C`之前，可能已经能够先按`Ctrl-V`，然后再按`Ctrl-C`输入`3`。 我们可以使用`IEXTEN`标志关闭此功能。

```c
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>
struct termios orig_termios;
void disableRawMode() { … }
void enableRawMode() {
  tcgetattr(STDIN_FILENO, &orig_termios);
  atexit(disableRawMode);
  struct termios raw = orig_termios;
  raw.c_iflag &= ~(IXON);
  raw.c_lflag &= ~(ECHO | ICANON | IEXTEN | ISIG);
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}
int main() { … }
```

`IEXTEN`来自`<termios.h>`，它是`I`开头但是不是input flag。

现在`Ctrl-V`可以被读作`22`,`Ctrl-O`可以被读作`15`.

### 修正`Ctrl-M`

`Ctrl-M`被读作`10`，但是我们希望读成`13`，同时`Ctrl-J`也被读成`10`。**Enter**也被读成`10`.终端把所有读入的回车符(`13`,'\r')全部转移为(`10`,'\n')。

```c
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>
struct termios orig_termios;
void disableRawMode() { … }
void enableRawMode() {
  tcgetattr(STDIN_FILENO, &orig_termios);
  atexit(disableRawMode);
  struct termios raw = orig_termios;
  raw.c_iflag &= ~(ICRNL | IXON);
  raw.c_lflag &= ~(ECHO | ICANON | IEXTEN | ISIG);
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}
int main() { … }
```

`ICRNL`来自`<termios.h>`，`I`表示input flag。`CR`代表carriage return，`NL`代表new line。

现在`Ctrl-M`可以被读作`13`，**Enter**也被读作`13`。

### 关闭所有输出处理

事实证明，终端在输出端进行了类似的转换。 它会将我们打印的每个换行符（“` \ n`”）转换为回车符，后跟换行符（“` \ r \ n`”）。 终端需要这两个字符才能开始新的一行文本。 回车键将光标移回当前行的开头，换行符将光标移至下一行，并在必要时滚动屏幕。（这两个不同的操作起源于打字机和电传打字时代。）

我们将通过关闭`OPOST`flag关闭输出处理。 默认情况下，“ `\ n`”到“` \ r \ n`”的转换可能是唯一打开的输出处理功能。 

```c
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>
struct termios orig_termios;
void disableRawMode() { … }
void enableRawMode() {
  tcgetattr(STDIN_FILENO, &orig_termios);
  atexit(disableRawMode);
  struct termios raw = orig_termios;
  raw.c_iflag &= ~(ICRNL | IXON);
  raw.c_oflag &= ~(OPOST);
  raw.c_lflag &= ~(ECHO | ICANON | IEXTEN | ISIG);
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}
int main() { … }
```

`OPOST`来自`<termios.h>`。`O`表示output flag.`POST`代表post-processing of output。

 如果现在运行该程序，会看到我们正在打印的换行符只是向下移动光标，而不是向屏幕左侧移动。  为了解决这个问题，我们将回车符添加到我们的`printf()`语句中。 

```c
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>
struct termios orig_termios;
void disableRawMode() { … }
void enableRawMode() { … }
int main() {
  enableRawMode();
  char c;
  while (read(STDIN_FILENO, &c, 1) == 1 && c != 'q') {
    if (iscntrl(c)) {
      printf("%d\r\n", c);
    } else {
      printf("%d ('%c')\r\n", c, c);
    }
  }
  return 0;
}
```

### 杂项flag

让我们关闭更多标志。

```c
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>
struct termios orig_termios;
void disableRawMode() { … }
void enableRawMode() {
  tcgetattr(STDIN_FILENO, &orig_termios);
  atexit(disableRawMode);
  struct termios raw = orig_termios;
  raw.c_iflag &= ~(BRKINT | ICRNL | INPCK | ISTRIP | IXON);
  raw.c_oflag &= ~(OPOST);
  raw.c_cflag |= (CS8);
  raw.c_lflag &= ~(ECHO | ICANON | IEXTEN | ISIG);
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}
int main() { … }
```

 `BRKINT`, `INPCK`, `ISTRIP`, 和 `CS8` 均来自 `<termios.h>`.

这一步可能对您没有任何效果，因为这些标志要么已经关闭，要么实际上不适用于现代终端仿真器。 但是在某个时候，关闭它们认为是启用“raw mode”的一部分，因此我们在程序中保留了传统。 

*  当`BRKINT`打开时，中断条件将导致`SIGINT`信号发送到程序，就像按`Ctrl-C`一样。 
* `INPCK`启用了奇偶校验，这似乎不适用于现代的终端仿真器。
* `ISTRIP`导致每个输入字节的第8位被剥离，这意味着它将设置为`0`。这可能已经关闭。
* `CS8`不是标志，它是一个具有多个位的位掩码，与我们要关闭的所有标志不同，我们使用按位或（`|`）运算符进行设置。它将字符大小（Character Size,CS）设置为每字节8位。在我的系统上，它已经设置好了。

 ### `read()`的超时

当前，`read()`将无限期地等待键盘输入，然后返回。 如果我们想在等待用户输入的同时做一些动画操作，该怎么办？ 我们可以设置一个超时，以便`read()`在一定时间内没有任何输入时返回。 

```c
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>
struct termios orig_termios;
void disableRawMode() { … }
void enableRawMode() {
  tcgetattr(STDIN_FILENO, &orig_termios);
  atexit(disableRawMode);
  struct termios raw = orig_termios;
  raw.c_iflag &= ~(BRKINT | ICRNL | INPCK | ISTRIP | IXON);
  raw.c_oflag &= ~(OPOST);
  raw.c_cflag |= (CS8);
  raw.c_lflag &= ~(ECHO | ICANON | IEXTEN | ISIG);
  raw.c_cc[VMIN] = 0;
  raw.c_cc[VTIME] = 1;
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}
int main() {
  enableRawMode();
  while (1) {
    char c = '\0';
    read(STDIN_FILENO, &c, 1);
    if (iscntrl(c)) {
      printf("%d\r\n", c);
    } else {
      printf("%d ('%c')\r\n", c, c);
    }
    if (c == 'q') break;
  }
  return 0;
}
```

`VMIN`和`VTIME`来自`<termios.h>`。它们是`c_cc`字段的索引，`c_cc`字段代表“控制字符（control Characters）”，即控制各种终端设置的字节数组。

 `VMIN`值设置`read()`返回之前所需的最小输入字节数。 我们将其设置为`0`，以便在有任何输入要读取时，`read()`会立即返回。 `VTIME`值设置`read()`返回之前要等待的最长时间。它以0.1秒为单位，因此我们将其设置为1/10秒或100毫秒。 

 运行该程序时，可以看到`read()`超时的频率。  如果您不提供任何输入，则`read()`会在不设置`c`变量的情况下返回，该变量保留其`0`值，因此会看到`0`被打印出来。 

如果在**Windows上使用Bash**，则可能会看到`read()`仍然无法输入。它似乎并不在乎`VTIME`值。幸运的是，这在我们的文本编辑器中不会有太大的不同，因为我们基本上还是会阻止输入。

### 错误处理

`enableRawMode()`现在使我们完全进入raw mode。现在该通过添加一些错误处理`die()`来打印错误信息退出程序。

```c
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>
struct termios orig_termios;
void die(const char *s) {
  perror(s);
  exit(1);
}
void disableRawMode() { … }
void enableRawMode() { … }
int main() { … }
```

`perror()`来自`<stdio.h>`，`exit()`来自`<stdlib.h>`.

大多数失败的C库函数都将设置全局`errno`变量以指示错误是什么。`perror()`查看全局`errno`变量并为其打印描述性错误消息。在打印错误消息之前，它还会打印给它的字符串，这是为了提供有关代码的哪一部分导致错误的上下文。 

 在打印出错误消息后，我们以退出状态`1`退出程序，该退出状态指示失败（任何非零值也是如此）。

 让我们检查每个库调用是否失败，并在失败时调用`die()`。

 ```c
#include <ctype.h>
#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>
struct termios orig_termios;
void die(const char *s) { … }
void disableRawMode() {
  if (tcsetattr(STDIN_FILENO, TCSAFLUSH, &orig_termios) == -1)
    die("tcsetattr");
}
void enableRawMode() {
  if (tcgetattr(STDIN_FILENO, &orig_termios) == -1) die("tcgetattr");
  atexit(disableRawMode);
  struct termios raw = orig_termios;
  raw.c_iflag &= ~(BRKINT | ICRNL | INPCK | ISTRIP | IXON);
  raw.c_oflag &= ~(OPOST);
  raw.c_cflag |= (CS8);
  raw.c_lflag &= ~(ECHO | ICANON | IEXTEN | ISIG);
  raw.c_cc[VMIN] = 0;
  raw.c_cc[VTIME] = 1;
  if (tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw) == -1) die("tcsetattr");
}
int main() {
  enableRawMode();
  while (1) {
    char c = '\0';
    if (read(STDIN_FILENO, &c, 1) == -1 && errno != EAGAIN) die("read");
    if (iscntrl(c)) {
      printf("%d\r\n", c);
    } else {
      printf("%d ('%c')\r\n", c, c);
    }
    if (c == 'q') break;
  }
  return 0;
}
 ```

`errno`和`EAGAIN`来自`<errno.h>`。

 `tcsetattr()`，`tcgetattr()`和`read()`均在失败时返回`-1`，并设置`errno`值以指示错误。 而不是像预期的那样仅返回0。为了使其在Cygwin中正常运行，我们不会将`EAGAIN`视为错误。 

 使`tcgetattr()`失败的一种简单方法是为程序提供文本文件或管道作为标准输入，而不是终端。  要为其提供文件作为标准输入，请运行`./kilo <kilo.c`。给它一个管道，`运行echo test | ./kilo`。两者都应导致来自`tcgetattr()`的相同错误，类似于` Inappropriate ioctl for device `。 

### 分段

即将结束本章有关进入raw mode的章节。我们现在要做的最后一件事是将代码分成几部分。这样可以使差异变得更短，因为差异中未更改的每个部分都将折叠为一行。

```c
/*** includes ***/
#include <ctype.h>
#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>
/*** data ***/
struct termios orig_termios;
/*** terminal ***/
void die(const char *s) { … }
void disableRawMode() { … }
void enableRawMode() { … }
/*** init ***/
int main() { … }
```

在下一章中，我们将进行一些更底层的终端输入/输出处理，并使用它来绘制到屏幕上，并允许用户来回移动光标。 

## Chapter 3 Raw输入输出

### 按下`Ctrl-Q`退出

 在上一章中，我们看到`Ctrl`键和字母键组合起来映射到1–26字节。我们可以使用它来检测`Ctrl`键组合并将其映射到编辑器中的不同操作。首先，将`Ctrl-Q`映射到退出操作。 

```c
/*** includes ***/
#include <ctype.h>
#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>
/*** defines ***/
#define CTRL_KEY(k) ((k) & 0x1f)
/*** data ***/
/*** terminal ***/
/*** init ***/
int main() {
  enableRawMode();
  while (1) {
    char c = '\0';
    if (read(STDIN_FILENO, &c, 1) == -1 && errno != EAGAIN) die("read");
    if (iscntrl(c)) {
      printf("%d\r\n", c);
    } else {
      printf("%d ('%c')\r\n", c, c);
    }
    if (c == CTRL_KEY('q')) break;
  }
  return 0;
}
```

`CTRL_KEY`宏按位与二进制值`00011111`的字符进行“与”运算。（在C语言中，通常使用十六进制指定位掩码，因为C没有二进制文字，并且一旦习惯使用十六进制，十六进制更简洁易读。） 换句话说，它将字符的高3位设置为`0`。  这反映了`Ctrl`键在终端中的功能：从`Ctrl`组合在一起按下的任何键中剥离第5位和第6位，并将其发送出去。（按照惯例，位编号从0开始。） ASCII字符集似乎是按这种方式设计的。（类似的设计可以设置和清除第5位以在小写和大写之间切换。） 

### 重构键盘输入

让我们创建一个用于低级按键读取的功能，以及另一个将按键映射到编辑器操作的功能。此时，我们还将停止打印按键。

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
void die(const char *s) { … }
void disableRawMode() { … }
void enableRawMode() { … }
char editorReadKey() {
  int nread;
  char c;
  while ((nread = read(STDIN_FILENO, &c, 1)) != 1) {
    if (nread == -1 && errno != EAGAIN) die("read");
  }
  return c;
}
/*** input ***/
void editorProcessKeypress() {
  char c = editorReadKey();
  switch (c) {
    case CTRL_KEY('q'):
      exit(0);
      break;
  }
}
/*** init ***/
int main() {
  enableRawMode();
  while (1) {
    editorProcessKeypress();
  }
  return 0;
}
```

`editorReadKey()`的工作是等待按键，然后将其返回。稍后，我们将扩展该功能以处理转义序列，这涉及读取代表单个按键的多个字节，如箭头键。

`editorProcessKeypress()`等待按键，然后对其进行处理。稍后，它将各种Ctrl键组合和其他特殊键映射到不同的编辑器功能，并将任何字母数字和其他可打印键的字符插入正在编辑的文本中。 

请注意，`editorReadKey()`属于`/ *** terminal *** /`部分，因为它处理低级终端输入，而`editorProcessKeypress()`属于新的`/ *** input *** /`部分，因为它处理将键映射到更高级别的编辑器功能。

现在我们大大简化了`main()`，我们将尝试保持这种方式。 

### 清屏

每次按键后，我们将在屏幕上呈现编辑器的用户界面。让我们从清除屏幕开始。

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
void die(const char *s) { … }
void disableRawMode() { … }
void enableRawMode() { … }
char editorReadKey() { … }
/*** output ***/
void editorRefreshScreen() {
  write(STDOUT_FILENO, "\x1b[2J", 4);
}
/*** input ***/
/*** init ***/
int main() {
  enableRawMode();
  while (1) {
    editorRefreshScreen();
    editorProcessKeypress();
  }
  return 0;
}
```

 `write()` 和 `STDOUT_FILENO` 来自 `<unistd.h>`.

`write()`调用中的`4`表示我们正在向终端写入`4`个字节。第一个字节是`\ x1b`，它是转义字符，或十进制的`27`。（尝试并记住`\ x1b`，我们将大量使用它。）其他三个字节为`[2J`。

我们正在向终端写一个转义序列。转义序列始终以转义字符（`27`）开头，后跟`[`字符。转义序列指示终端执行各种文本格式化任务，例如为**文本着色**，**四处移动光标**以及**清除屏幕**的一部分。 

 我们正在使用J命令（ [Erase In Display](http://vt100.net/docs/vt100-ug/chapter3.html#ED) ）来清除屏幕。转义序列命令带有参数，该参数位于命令之前。 在这种情况下，参数为2，表示要清除整个屏幕。  `<esc> [1J`将清除屏幕直到光标所在的位置，而`<esc> [0J`将清除屏幕从光标直到屏幕结尾的位置。 另外，`0`是J的默认参数，因此仅`<esc> [J`本身也可以将屏幕从光标到末尾清除。

 对于我们的文本编辑器，我们将主要使用VT100转义序列，现代终端仿真器广泛支持VT100转义序列 。有关每个转义序列的完整文档，请参阅《 [VT100 User Guide](http://vt100.net/docs/vt100-ug/chapter3.html) 》。

如果要支持最大数量的终端，可以使用 [ncurses](https://en.wikipedia.org/wiki/Ncurses)库，该库使用[terminfo](https://en.wikipedia.org/wiki/Terminfo)数据库来确定终端的功能以及该特定终端要使用的转义序列。

### 重新定位光标

 您可能会注意到`<esc> [2J`命令将光标移到了屏幕底部。让我们将其重新放置在左上角，以便我们准备从上到下绘制编辑器界面。 

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
/*** output ***/
void editorRefreshScreen() {
  write(STDOUT_FILENO, "\x1b[2J", 4);
  write(STDOUT_FILENO, "\x1b[H", 3);
}
/*** input ***/
/*** init ***/
```

此转义序列只有`3`个字节长，并使用`H`命令 ([Cursor Position](http://vt100.net/docs/vt100-ug/chapter3.html#CUP)) 定位光标.` H`命令实际上接受两个参数：将光标定位在的行号和列号。因此，如果您有一个`80×24`尺寸的终端，并且希望光标位于屏幕中央，则可以使用命令`<esc> [12;40H`.（多个参数用;字符分隔。） H的默认参数都恰好是`1`，因此我们可以将两个参数都保留掉，它将把光标定位在第一行和第一列，就像我们已经发送了`<esc> [1; 1H`命令一样。（行和列的编号从`1`开始，而不是`0`。）

### 退出时清屏

退出程序后，让我们清除屏幕并重新定位光标。如果在渲染屏幕的过程中发生错误，我们不希望在屏幕上留下一堆垃圾，并且我们不希望该错误在光标碰巧出现的任何地方被打印出来。

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
void die(const char *s) {
  write(STDOUT_FILENO, "\x1b[2J", 4);
  write(STDOUT_FILENO, "\x1b[H", 3);
  perror(s);
  exit(1);
}
void disableRawMode() { … }
void enableRawMode() { … }
char editorReadKey() { … }
/*** output ***/
/*** input ***/
void editorProcessKeypress() {
  char c = editorReadKey();
  switch (c) {
    case CTRL_KEY('q'):
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
  }
}
/*** init ***/
```

我们有两个要清除屏幕的退出点：`die()`，以及当用户按下`Ctrl-Q`退出时。

我们可以在程序退出时使用`atexit()`来清除屏幕，但随后`die()`打印的错误消息将在打印后立即消除。

### 波浪号

现在该开始绘制了。让我们在屏幕的左侧绘制波浪号（`~`）列，就像vim一样。在文本编辑器中，我们将在要编辑的文件末尾之后的所有行的开头绘制波浪号。

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
/*** output ***/
void editorDrawRows() {
  int y;
  for (y = 0; y < 24; y++) {
    write(STDOUT_FILENO, "~\r\n", 3);
  }
}
void editorRefreshScreen() {
  write(STDOUT_FILENO, "\x1b[2J", 4);
  write(STDOUT_FILENO, "\x1b[H", 3);
  editorDrawRows();
  write(STDOUT_FILENO, "\x1b[H", 3);
}
/*** input ***/
/*** init ***/
```

`editorDrawRows()`将处理绘制正在编辑的文本缓冲区的每一行。目前，它在每行中都显示一个波浪号，这意味着该行不属于文件，并且不能包含任何文本。 

我们尚不知道终端的大小，因此我们不知道要绘制多少行。现在，我们只绘制`24`行。 

完成绘制后，我们执行另一个`<esc> [H`转义序列以将光标重新定位在左上角。 

### 全局状态

我们的下一个目标是获取终端的大小，因此我们知道在`editorDrawRows()`中绘制多少行。但首先，让我们设置一个包含我们的编辑器状态的全局结构，该状态将用于存储终端的宽度和高度。现在，让我们将`orig_termios`全局变量放入结构中。

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
struct editorConfig {
  struct termios orig_termios;
};
struct editorConfig E;
/*** terminal ***/
void die(const char *s) { … }
void disableRawMode() {
  if (tcsetattr(STDIN_FILENO, TCSAFLUSH, &E.orig_termios) == -1)
    die("tcsetattr");
}
void enableRawMode() {
  if (tcgetattr(STDIN_FILENO, &E.orig_termios) == -1) die("tcgetattr");
  atexit(disableRawMode);
  struct termios raw = E.orig_termios;
  raw.c_iflag &= ~(BRKINT | ICRNL | INPCK | ISTRIP | IXON);
  raw.c_oflag &= ~(OPOST);
  raw.c_cflag |= (CS8);
  raw.c_lflag &= ~(ECHO | ICANON | IEXTEN | ISIG);
  raw.c_cc[VMIN] = 0;
  raw.c_cc[VTIME] = 1;
  if (tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw) == -1) die("tcsetattr");
}
char editorReadKey() { … }
/*** output ***/
/*** input ***/
/*** init ***/
```

 包含编辑器状态的全局变量名为E。我们必须用`E.orig_termios`替换所有出现的`orig_termios`。 

###  窗口大小，简单的方法 

在大多数系统上，能够通过使用`TIOCGWINSZ`请求简单地调用`ioctl()`来获得终端的大小。(  **T**erminal **IOC**tl (which itself stands for **I**nput/**O**utput **C**on**t**ro**l**) **G**et **WIN**dow **S**i**Z**e. )

```c
/*** includes ***/
#include <ctype.h>
#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/ioctl.h>
#include <termios.h>
#include <unistd.h>
/*** defines ***/
/*** data ***/
/*** terminal ***/
void die(const char *s) { … }
void disableRawMode() { … }
void enableRawMode() { … }
char editorReadKey() { … }
int getWindowSize(int *rows, int *cols) {
  struct winsize ws;
  if (ioctl(STDOUT_FILENO, TIOCGWINSZ, &ws) == -1 || ws.ws_col == 0) {
    return -1;
  } else {
    *cols = ws.ws_col;
    *rows = ws.ws_row;
    return 0;
  }
}
/*** output ***/
/*** input ***/
/*** init ***/
```

 `ioctl()`, `TIOCGWINSZ`, 和 `struct winsize`均来自`<sys/ioctl.h>`. 

 成功后，`ioctl()`会将终端的列数宽和行数高处放入给定的`winsize`结构中。失败时，`ioctl()`返回`-1`。我们还检查以确保返回的值不为`0`，因为显然这可能是错误的结果。 如果`ioctl()`以任何一种方式失败，我们将通过返回`-1`来使`getWindowSize()`报告失败。 如果成功，则通过设置传递给函数的int引用将值传递回去。（这是使函数在C中返回多个值的常用方法。它还允许您使用返回值指示成功或失败。） 

现在，我们将`screenrows`和`screencols`添加到我们的全局编辑器状态，然后调用`getWindowSize()`来填充这些值。

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
struct editorConfig {
  int screenrows;
  int screencols;
  struct termios orig_termios;
};
struct editorConfig E;
/*** terminal ***/
/*** output ***/
/*** input ***/
/*** init ***/
void initEditor() {
  if (getWindowSize(&E.screenrows, &E.screencols) == -1) die("getWindowSize");
}
int main() {
  enableRawMode();
  initEditor();
  while (1) {
    editorRefreshScreen();
    editorProcessKeypress();
  }
  return 0;
}
```

 `initEditor()`的工作将是初始化E结构中的所有字段。 

现在，我们准备在屏幕上显示正确的波浪号：

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
/*** output ***/
void editorDrawRows() {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    write(STDOUT_FILENO, "~\r\n", 3);
  }
}
void editorRefreshScreen() { … }
/*** input ***/
/*** init ***/
```

### 窗口大小，困难的方法

无法保证`ioctl()`能够在所有系统上请求窗口大小，因此提供一种获取窗口大小的备用方法。 

策略是将光标定位在屏幕的右下角，然后使用转义序列让我们查询光标的位置。这告诉我们屏幕上必须有多少行和几列。

首先，将光标移到右下角。

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
void die(const char *s) { … }
void disableRawMode() { … }
void enableRawMode() { … }
char editorReadKey() { … }
int getWindowSize(int *rows, int *cols) {
  struct winsize ws;
  if (1 || ioctl(STDOUT_FILENO, TIOCGWINSZ, &ws) == -1 || ws.ws_col == 0) {
    if (write(STDOUT_FILENO, "\x1b[999C\x1b[999B", 12) != 12) return -1;
    editorReadKey();
    return -1;
  } else {
    *cols = ws.ws_col;
    *rows = ws.ws_row;
    return 0;
  }
}
/*** output ***/
/*** input ***/
/*** init ***/
```

正如您可能从代码中看到的那样，没有简单的“将光标移至右下角”命令。

我们依次发送两个转义序列。`C`命令（向前光标）将光标向右移动，而`B`命令（向下光标）将光标向下移动。属性说明了将其向右或向下移动多少。我们使用一个非常大的值999，该值应确保光标到达屏幕的右边缘和下边缘。

`C`和`B`命令专门记录了指令，以阻止光标越过屏幕边缘。我们之所以不使用`<esc> [999; 999H`命令，是因为该文档没有指定尝试将光标移到屏幕外时会发生的情况。 

请注意，我们加上`1 ||`。暂时置于if条件的最前面，以便我们可以测试我们正在开发的此后备分支。

由于此时我们总是从`getWindowSize()`返回`-1`（表示发生错误），因此我们对`editorReadKey()`进行了调用，因此我们可以在程序调用`die()`之前观察转义序列的结果，并清除屏幕。运行该程序时，您应该看到光标位于屏幕的右下角，然后当您按下某个键时，它会在清除屏幕后看到`die()`打印的错误消息。

接下来，我们需要获取光标位置.`n`命令（设备状态报告）可用于向终端查询状态信息。给它一个参数6来要求光标位置。 然后，我们可以从标准输入中读取答复。 让我们从标准输入中打印出每个字符，以查看回复的样子。 

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
void die(const char *s) { … }
void disableRawMode() { … }
void enableRawMode() { … }
char editorReadKey() { … }
int getCursorPosition(int *rows, int *cols) {
  if (write(STDOUT_FILENO, "\x1b[6n", 4) != 4) return -1;
  printf("\r\n");
  char c;
  while (read(STDIN_FILENO, &c, 1) == 1) {
    if (iscntrl(c)) {
      printf("%d\r\n", c);
    } else {
      printf("%d ('%c')\r\n", c, c);
    }
  }
  editorReadKey();
  return -1;
}
int getWindowSize(int *rows, int *cols) {
  struct winsize ws;
  if (1 || ioctl(STDOUT_FILENO, TIOCGWINSZ, &ws) == -1 || ws.ws_col == 0) {
    if (write(STDOUT_FILENO, "\x1b[999C\x1b[999B", 12) != 12) return -1;
    return getCursorPosition(rows, cols);
  } else {
    *cols = ws.ws_col;
    *rows = ws.ws_row;
    return 0;
  }
}
/*** output ***/
/*** input ***/
/*** init ***/
```

答复是一个转义序列！它是一个转义字符（`27`），后跟`[`字符，然后是实际响应：`24; 80R`，或类似的字符。（此转义序列记录为“光标位置报告”。）

和以前一样，我们已经插入了对`editorReadKey()`的临时调用，以便我们在退出屏幕清除之前观察调试输出。 

（注意：如果您在**Windows上使用Bash**，则`read()`不会超时，因此您将陷入无限循环。您必须在外部终止进程，或者退出并重新打开命令提示符窗口。）

我们将不得不解析此响应。但首先，让我们将其读入缓冲区。我们将继续读取字符，直到到达`R`字符为止。

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
void die(const char *s) { … }
void disableRawMode() { … }
void enableRawMode() { … }
char editorReadKey() { … }
int getCursorPosition(int *rows, int *cols) {
  char buf[32];
  unsigned int i = 0;
  if (write(STDOUT_FILENO, "\x1b[6n", 4) != 4) return -1;
  while (i < sizeof(buf) - 1) {
    if (read(STDIN_FILENO, &buf[i], 1) != 1) break;
    if (buf[i] == 'R') break;
    i++;
  }
  buf[i] = '\0';
  printf("\r\n&buf[1]: '%s'\r\n", &buf[1]);
  editorReadKey();
  return -1;
}
int getWindowSize(int *rows, int *cols) { … }
/*** output ***/
/*** input ***/
/*** init ***/
```

当我们打印缓冲区时，我们不想打印`\ x1b`字符，因为终端会将其解释为转义序列，而不显示它。因此，我们通过将`＆buf [1]`传递给`printf()`来跳过`buf`中的第一个字符。` printf()`期望字符串以`0`字节结尾，因此我们确保将`'\ 0'`分配给buf的最后一个字节。 

如果运行该程序，您会看到我们以`buf`的形式出现`<esc> [24; 80`。使用`sscanf()`解析两个数字： 

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
void die(const char *s) { … }
void disableRawMode() { … }
void enableRawMode() { … }
char editorReadKey() { … }
int getCursorPosition(int *rows, int *cols) {
  char buf[32];
  unsigned int i = 0;
  if (write(STDOUT_FILENO, "\x1b[6n", 4) != 4) return -1;
  while (i < sizeof(buf) - 1) {
    if (read(STDIN_FILENO, &buf[i], 1) != 1) break;
    if (buf[i] == 'R') break;
    i++;
  }
  buf[i] = '\0';
  if (buf[0] != '\x1b' || buf[1] != '[') return -1;
  if (sscanf(&buf[2], "%d;%d", rows, cols) != 2) return -1;
  return 0;
}
int getWindowSize(int *rows, int *cols) { … }
/*** output ***/
/*** input ***/
/*** init ***/
```

`sscanf()`来自`<stdio.h>`.

首先，我们确保它以转义序列响应。然后，我们将指向`buf`的第三个字符的指针传递给`sscanf()`，跳过`“ \ x1b”`和`“ [”`字符。我们还将字符串`％d;％d`传递给它，该字符串告诉它解析两个用`;`分隔的整数，并将值放入`rows`和`cols`变量中。

现在，我们用于获取窗口大小的后备方法已经完成。看到`editorDrawRows()`根据终端的高度打印了正确的波浪号。 

现在我们知道了，可以删除`1 ||`。我们暂时设置了`if`条件。 

### 最后一行

也许您注意到屏幕的最后一行似乎没有波浪号。那是因为我们的代码中有一个小错误。当我们打印最终的波浪号时，然后像在其他任何行上一样打印`“ \ r \ n”`，但这会导致终端滚动以便为新的空白行腾出空间。让我们在打印`“ \ r \ n”`时将最后一行作为例外。

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
/*** output ***/
void editorDrawRows() {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    write(STDOUT_FILENO, "~", 1);
    if (y < E.screenrows - 1) {
      write(STDOUT_FILENO, "\r\n", 2);
    }
  }
}
void editorRefreshScreen() { … }
/*** input ***/
/*** init ***/
```

### 追加缓冲区

每次刷新屏幕时都进行一堆小的`write()`并不是一个好主意。最好做一个大的`write()`，以确保整个屏幕立即更新。否则，`write()`之间可能会有小的无法预测的停顿，这将导致令人讨厌的闪烁效果。 

我们想用将字符串追加到缓冲区的代码替换所有`write()`调用，然后最后将缓冲区`write()`。不幸的是，C没有动态字符串，因此我们将创建自己的动态字符串类型，该类型支持一种操作：追加。

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
void die(const char *s) { … }
void disableRawMode() { … }
void enableRawMode() { … }
char editorReadKey() { … }
int getCursorPosition(int *rows, int *cols) { … }
int getWindowSize(int *rows, int *cols) { … }
/*** append buffer ***/
struct abuf {
  char *b;
  int len;
};
#define ABUF_INIT {NULL, 0}
/*** output ***/
/*** input ***/
/*** init ***/
```

追加缓冲区由指向内存中缓冲区的指针和长度组成。我们定义一个`ABUF_INIT`常量，该常量表示一个空缓冲区。这充当我们`abuf`类型的构造函数。 

接下来，让我们定义`abAppend()`操作以及`abFree()`析构函数。

```c
/*** includes ***/
#include <ctype.h>
#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/ioctl.h>
#include <termios.h>
#include <unistd.h>
/*** defines ***/
/*** data ***/
/*** terminal ***/
/*** append buffer ***/
struct abuf { … };
#define ABUF_INIT {NULL, 0}
void abAppend(struct abuf *ab, const char *s, int len) {
  char *new = realloc(ab->b, ab->len + len);
  if (new == NULL) return;
  memcpy(&new[ab->len], s, len);
  ab->b = new;
  ab->len += len;
}
void abFree(struct abuf *ab) {
  free(ab->b);
}
/*** output ***/
/*** input ***/
/*** init ***/
```

`realloc()` 和 `free()`来自`<stdlib.h>`. `memcpy()`来自`<string.h>`. 

要将字符串`s`附加到`abuf`，我们要做的第一件事是确保分配足够的内存来容纳新字符串。我们要求`realloc()`给我们一个内存块，它是当前字符串的大小加上我们要追加的字符串的大小。`realloc()`将扩展我们已经分配的内存块的大小，或者将`free()`释放当前的内存块并在足以容纳新字符串的其他地方分配新的内存块。

然后，我们使用`memcpy()`将当前数据结束后的字符串`s`复制到缓冲区中，然后将`abuf`的指针和长度更新为新值。 

`abFree()`是一个析构函数，用于释放`abuf`使用的动态内存。`abuf`类型随时可以使用。 

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
/*** append buffer ***/
/*** output ***/
void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    abAppend(ab, "~", 1);
    if (y < E.screenrows - 1) {
      abAppend(ab, "\r\n", 2);
    }
  }
}
void editorRefreshScreen() {
  struct abuf ab = ABUF_INIT;
  abAppend(&ab, "\x1b[2J", 4);
  abAppend(&ab, "\x1b[H", 3);
  editorDrawRows(&ab);
  abAppend(&ab, "\x1b[H", 3);
  write(STDOUT_FILENO, ab.b, ab.len);
  abFree(&ab);
}
/*** input ***/
/*** init ***/
```

在`editorRefreshScreen()`中，我们首先通过分配`ABUF_INIT`来初始化一个名为`ab`的新`abuf`。然后，我们用`abAppend(&ab,...)`替换每次出现的`write(STDOUT_FILENO,...)`。我们还将`ab`传递给`editorDrawRows()`，因此它也可以使用`abAppend（）`。最后，我们将缓冲区的内容`write()`到标准输出中，并释放`abuf`使用的内存。

###  重绘时隐藏光标 

烦人的闪烁效果还有另一个可能的来源，我们现在要解决。终端绘制到屏幕时，光标可能会在屏幕中间的某处显示一秒钟。为了确保不会发生这种情况，请在刷新屏幕之前先隐藏光标，然后在刷新完成后立即再次显示。 

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
/*** append buffer ***/
/*** output ***/
void editorDrawRows(struct abuf *ab) { … }
void editorRefreshScreen() {
  struct abuf ab = ABUF_INIT;
  abAppend(&ab, "\x1b[?25l", 6);
  abAppend(&ab, "\x1b[2J", 4);
  abAppend(&ab, "\x1b[H", 3);
  editorDrawRows(&ab);
  abAppend(&ab, "\x1b[H", 3);
  abAppend(&ab, "\x1b[?25h", 6);
  write(STDOUT_FILENO, ab.b, ab.len);
  abFree(&ab);
}
/*** input ***/
/*** init ***/
```

我们使用转义序列来告诉终端隐藏并显示光标。`h`和`l`命令（设置模式，复位模式）用于打开和关闭各种终端功能或“模式”。 刚刚链接到的《 VT100用户指南》并未记录我们上面使用的参数`?25`。在以后的VT模型中出现了光标隐藏/显示功能。因此，某些终端可能不支持隐藏/显示光标，但是如果不支持，则它们只会忽略这些转义序列，在这种情况下，这没什么大不了的。 

### 一次清除一行

与其在每次刷新之前清除整个屏幕，不如在重新绘制它们时清除每一行似乎更为理想。让我们删除`<esc> [2J`（清除整个屏幕）转义序列，然后在我们绘制的每行末尾放置一个`<esc> [K`序列。 

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
/*** append buffer ***/
/*** output ***/
void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    abAppend(ab, "~", 1);
    abAppend(ab, "\x1b[K", 3);
    if (y < E.screenrows - 1) {
      abAppend(ab, "\r\n", 2);
    }
  }
}
void editorRefreshScreen() {
  struct abuf ab = ABUF_INIT;
  abAppend(&ab, "\x1b[?25l", 6);
  abAppend(&ab, "\x1b[2J", 4);
  abAppend(&ab, "\x1b[H", 3);
  editorDrawRows(&ab);
  abAppend(&ab, "\x1b[H", 3);
  abAppend(&ab, "\x1b[?25h", 6);
  write(STDOUT_FILENO, ab.b, ab.len);
  abFree(&ab);
}
/*** input ***/
/*** init ***/
```

`K`命令（在线擦除）擦除当前行的一部分。 其参数类似于`J`命令的参数：`2`擦除整行，`1`擦除光标左侧的行的一部分，`0`擦除光标右侧的行的一部分。  `0`是默认参数，这就是我们想要的，因此我们省略该参数，而只使用`<esc> [K`。 

### 欢迎信息

也许是时候显示欢迎信息了。让我们在屏幕下方三分之一处显示编辑器的名称和版本号。 

```c
/*** includes ***/
/*** defines ***/
#define KILO_VERSION "0.0.1"
#define CTRL_KEY(k) ((k) & 0x1f)
/*** data ***/
/*** terminal ***/
/*** append buffer ***/
/*** output ***/
void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    if (y == E.screenrows / 3) {
      char welcome[80];
      int welcomelen = snprintf(welcome, sizeof(welcome),
        "Kilo editor -- version %s", KILO_VERSION);
      if (welcomelen > E.screencols) welcomelen = E.screencols;
      abAppend(ab, welcome, welcomelen);
    } else {
      abAppend(ab, "~", 1);
    }
    abAppend(ab, "\x1b[K", 3);
    if (y < E.screenrows - 1) {
      abAppend(ab, "\r\n", 2);
    }
  }
}
void editorRefreshScreen() { … }
/*** input ***/
/*** init ***/
```

`snprintf()`来自`<stdio.h>`.

我们使用`welcome`缓冲区和`snprintf()`将`KILO_VERSION`字符串插入到欢迎消息中。如果终端太小而无法容纳我们的欢迎信息，我们还将截断字符串的长度。 

下面来居中它：

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
/*** append buffer ***/
/*** output ***/
void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    if (y == E.screenrows / 3) {
      char welcome[80];
      int welcomelen = snprintf(welcome, sizeof(welcome),
        "Kilo editor -- version %s", KILO_VERSION);
      if (welcomelen > E.screencols) welcomelen = E.screencols;
      int padding = (E.screencols - welcomelen) / 2;
      if (padding) {
        abAppend(ab, "~", 1);
        padding--;
      }
      while (padding--) abAppend(ab, " ", 1);
      abAppend(ab, welcome, welcomelen);
    } else {
      abAppend(ab, "~", 1);
    }
    abAppend(ab, "\x1b[K", 3);
    if (y < E.screenrows - 1) {
      abAppend(ab, "\r\n", 2);
    }
  }
}
void editorRefreshScreen() { … }
/*** input ***/
/*** init ***/
```

要使字符串居中，请将屏幕宽度除以`2`，然后从中减去`字符串长度的一半`。换句话说：`E.screencols / 2-welcomelen / 2`，简化为`(E.screencols-welcomelen)/ 2`。它告诉您应该从屏幕的左边缘开始打印多行。因此，我们用空格字符填充该空间，但第一个字符除外，第一个字符应为~。

### 移动光标

现在让我们专注于输入。我们希望用户能够移动光标。第一步是在全局编辑器状态下跟踪光标的`x`和`y`位置。 

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
struct editorConfig {
  int cx, cy;
  int screenrows;
  int screencols;
  struct termios orig_termios;
};
struct editorConfig E;
/*** terminal ***/
/*** append buffer ***/
/*** output ***/
/*** input ***/
/*** init ***/
void initEditor() {
  E.cx = 0;
  E.cy = 0;
  if (getWindowSize(&E.screenrows, &E.screencols) == -1) die("getWindowSize");
}
int main() { … }
```

`E.cx`是光标的水平坐标（列），`E.cy`是垂直坐标（行）。我们希望将它们都初始化为`0`，因为我们希望光标从屏幕的左上角开始。（由于C语言使用从0开始的索引，因此我们将尽可能使用0索引的值。）

现在，将代码添加到`editorRefreshScreen()`中，以将光标移动到`E.cx`和`E.cy`中存储的位置。

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
/*** append buffer ***/
/*** output ***/
void editorDrawRows(struct abuf *ab) { … }
void editorRefreshScreen() {
  struct abuf ab = ABUF_INIT;
  abAppend(&ab, "\x1b[?25l", 6);
  abAppend(&ab, "\x1b[H", 3);
  editorDrawRows(&ab);
  char buf[32];
  snprintf(buf, sizeof(buf), "\x1b[%d;%dH", E.cy + 1, E.cx + 1);
  abAppend(&ab, buf, strlen(buf));
  abAppend(&ab, "\x1b[?25h", 6);
  write(STDOUT_FILENO, ab.b, ab.len);
  abFree(&ab);
}
/*** input ***/
/*** init ***/
```

我们将旧的`H`命令更改为带有参数的`H`命令，指定了我们希望光标移动到的确切位置。（请确保您删除了旧的H命令）

我们在`E.cy`和`E.cx`上加1，以将`0`索引值转换为终端使用的`1`索引值。

此时，您可以尝试将`E.cx`初始化为`10`或类似的值，或者将`E.cx ++`插入主循环中，以确认代码到目前为止是否可以正常工作。

接下来，我们将允许用户使用wasd键移动光标。（如果您不熟悉将这些键用作箭头键：w是向上箭头，s是向下箭头，a是左箭头，d是右箭头。）

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
/*** append buffer ***/
/*** output ***/
/*** input ***/
void editorMoveCursor(char key) {
  switch (key) {
    case 'a':
      E.cx--;
      break;
    case 'd':
      E.cx++;
      break;
    case 'w':
      E.cy--;
      break;
    case 's':
      E.cy++;
      break;
  }
}
void editorProcessKeypress() {
  char c = editorReadKey();
  switch (c) {
    case CTRL_KEY('q'):
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case 'w':
    case 's':
    case 'a':
    case 'd':
      editorMoveCursor(c);
      break;
  }
}
/*** init ***/
```

 现在，应该能够使用这些键在光标周围移动。

### 方向键

 现在，我们有了一种映射按键来移动光标的方法，让我们用箭头键替换wasd键。  在上一章中，我们看到按箭头键会将多个字节作为输入发送到我们的程序。 这些字节采用转义序列的形式，以`'\ x1b'`，`'['`，然后是`'A'`，`'B'`，`'C'`或`'D'`开头，具体取决于四个箭头键中的哪个按下。让我们修改`editorReadKey()`以一次按下该键即可读取这种形式的转义序列。 

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
void die(const char *s) { … }
void disableRawMode() { … }
void enableRawMode() { … }
char editorReadKey() {
  int nread;
  char c;
  while ((nread = read(STDIN_FILENO, &c, 1)) != 1) {
    if (nread == -1 && errno != EAGAIN) die("read");
  }
  if (c == '\x1b') {
    char seq[3];
    if (read(STDIN_FILENO, &seq[0], 1) != 1) return '\x1b';
    if (read(STDIN_FILENO, &seq[1], 1) != 1) return '\x1b';
    if (seq[0] == '[') {
      switch (seq[1]) {
        case 'A': return 'w';
        case 'B': return 's';
        case 'C': return 'd';
        case 'D': return 'a';
      }
    }
    return '\x1b';
  } else {
    return c;
  }
}
int getCursorPosition(int *rows, int *cols) { … }
int getWindowSize(int *rows, int *cols) { … }
/*** append buffer ***/
/*** output ***/
/*** input ***/
/*** init ***/
```

如果读取转义字符，则立即将另外两个字节读取到`seq`缓冲区中。如果这些读取中的任何一个超时（在0.1秒后），那么我们假定用户只是按下了`Escape`键并返回该键。否则，我们将查看转义序列是否为箭头键转义序列。如果是的话，我们暂时返回相应的`wasd`字符。如果不是我们识别出的转义序列，我们只需返回转义字符。 

我们将`seq`缓冲区设置为`3`个字节长，因为将来我们将处理更长的转义序列。 

我们基本上已经将箭头键别名化为wasd键。这样可以使箭头键立即工作，但将wasd键仍映射到`editorMoveCursor()`函数。我们想要的是`editorReadKey()`为每个箭头键返回特殊值，以使我们能够确定已按下特定的箭头键。 

首先，用常数`ARROW_UP`，`ARROW_LEFT`，`ARROW_DOWN`和`ARROW_RIGHT`替换wasd字符的每个实例。 

```c
/*** includes ***/
/*** defines ***/
#define KILO_VERSION "0.0.1"
#define CTRL_KEY(k) ((k) & 0x1f)
enum editorKey {
  ARROW_LEFT = 'a',
  ARROW_RIGHT = 'd',
  ARROW_UP = 'w',
  ARROW_DOWN = 's'
};
/*** data ***/
/*** terminal ***/
void die(const char *s) { … }
void disableRawMode() { … }
void enableRawMode() { … }
char editorReadKey() {
  int nread;
  char c;
  while ((nread = read(STDIN_FILENO, &c, 1)) != 1) {
    if (nread == -1 && errno != EAGAIN) die("read");
  }
  if (c == '\x1b') {
    char seq[3];
    if (read(STDIN_FILENO, &seq[0], 1) != 1) return '\x1b';
    if (read(STDIN_FILENO, &seq[1], 1) != 1) return '\x1b';
    if (seq[0] == '[') {
      switch (seq[1]) {
        case 'A': return ARROW_UP;
        case 'B': return ARROW_DOWN;
        case 'C': return ARROW_RIGHT;
        case 'D': return ARROW_LEFT;
      }
    }
    return '\x1b';
  } else {
    return c;
  }
}
int getCursorPosition(int *rows, int *cols) { … }
int getWindowSize(int *rows, int *cols) { … }
/*** append buffer ***/
/*** output ***/
/*** input ***/
void editorMoveCursor(char key) {
  switch (key) {
    case ARROW_LEFT:
      E.cx--;
      break;
    case ARROW_RIGHT:
      E.cx++;
      break;
    case ARROW_UP:
      E.cy--;
      break;
    case ARROW_DOWN:
      E.cy++;
      break;
  }
}
void editorProcessKeypress() {
  char c = editorReadKey();
  switch (c) {
    case CTRL_KEY('q'):
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case ARROW_UP:
    case ARROW_DOWN:
    case ARROW_LEFT:
    case ARROW_RIGHT:
      editorMoveCursor(c);
      break;
  }
}
/*** init ***/
```

现在，我们只需要在`editorKey`枚举中选择与`wasd`不冲突的箭头键表示即可。我们将为他们提供一个大的整数值，该值超出了`char`的范围，这样它们就不会与任何普通的按键冲突。我们还必须将所有存储按键的变量更改为`int`类型而不是`char`类型。 

```c
arrow-keys-int
/*** includes ***/
/*** defines ***/
#define KILO_VERSION "0.0.1"
#define CTRL_KEY(k) ((k) & 0x1f)
enum editorKey {
  ARROW_LEFT = 1000,
  ARROW_RIGHT,
  ARROW_UP,
  ARROW_DOWN
};
/*** data ***/
/*** terminal ***/
void die(const char *s) { … }
void disableRawMode() { … }
void enableRawMode() { … }
int editorReadKey() {
  int nread;
  char c;
  while ((nread = read(STDIN_FILENO, &c, 1)) != 1) {
    if (nread == -1 && errno != EAGAIN) die("read");
  }
  if (c == '\x1b') {
    char seq[3];
    if (read(STDIN_FILENO, &seq[0], 1) != 1) return '\x1b';
    if (read(STDIN_FILENO, &seq[1], 1) != 1) return '\x1b';
    if (seq[0] == '[') {
      switch (seq[1]) {
        case 'A': return ARROW_UP;
        case 'B': return ARROW_DOWN;
        case 'C': return ARROW_RIGHT;
        case 'D': return ARROW_LEFT;
      }
    }
    return '\x1b';
  } else {
    return c;
  }
}
int getCursorPosition(int *rows, int *cols) { … }
int getWindowSize(int *rows, int *cols) { … }
/*** append buffer ***/
/*** output ***/
/*** input ***/
void editorMoveCursor(int key) {
  switch (key) {
    case ARROW_LEFT:
      E.cx--;
      break;
    case ARROW_RIGHT:
      E.cx++;
      break;
    case ARROW_UP:
      E.cy--;
      break;
    case ARROW_DOWN:
      E.cy++;
      break;
  }
}
void editorProcessKeypress() {
  int c = editorReadKey();
  switch (c) {
    case CTRL_KEY('q'):
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case ARROW_UP:
    case ARROW_DOWN:
    case ARROW_LEFT:
    case ARROW_RIGHT:
      editorMoveCursor(c);
      break;
  }
}
/*** init ***/
```

通过将枚举中的第一个常数设置为`1000`，其余常数将获得递增值`1001`、`1002`、`1003`，依此类推。

### 防止光标超出屏幕

当前，您可以使`E.cx`和`E.cy`值变为负值，或者超过屏幕的右边缘和下边缘。让我们通过在`editorMoveCursor()`中进行一些边界检查来避免这种情况。

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
/*** append buffer ***/
/*** output ***/
/*** input ***/
void editorMoveCursor(int key) {
  switch (key) {
    case ARROW_LEFT:
      if (E.cx != 0) {
        E.cx--;
      }
      break;
    case ARROW_RIGHT:
      if (E.cx != E.screencols - 1) {
        E.cx++;
      }
      break;
    case ARROW_UP:
      if (E.cy != 0) {
        E.cy--;
      }
      break;
    case ARROW_DOWN:
      if (E.cy != E.screenrows - 1) {
        E.cy++;
      }
      break;
  }
}
void editorProcessKeypress() { … }
/*** init ***/
```

### `Page Up`和`Page Down`

为了完成我们的底层终端代码，我们需要检测更多使用转义序列的特殊按键，如箭头键一样。我们将从Page Up和Page Down键开始。Page Up发送为`<esc> [5〜`，Page Down发送为`<esc> [6〜`。

 ```c
/*** includes ***/
/*** defines ***/
#define KILO_VERSION "0.0.1"
#define CTRL_KEY(k) ((k) & 0x1f)
enum editorKey {
  ARROW_LEFT = 1000,
  ARROW_RIGHT,
  ARROW_UP,
  ARROW_DOWN,
  PAGE_UP,
  PAGE_DOWN
};
/*** data ***/
/*** terminal ***/
void die(const char *s) { … }
void disableRawMode() { … }
void enableRawMode() { … }
int editorReadKey() {
  int nread;
  char c;
  while ((nread = read(STDIN_FILENO, &c, 1)) != 1) {
    if (nread == -1 && errno != EAGAIN) die("read");
  }
  if (c == '\x1b') {
    char seq[3];
    if (read(STDIN_FILENO, &seq[0], 1) != 1) return '\x1b';
    if (read(STDIN_FILENO, &seq[1], 1) != 1) return '\x1b';
    if (seq[0] == '[') {
      if (seq[1] >= '0' && seq[1] <= '9') {
        if (read(STDIN_FILENO, &seq[2], 1) != 1) return '\x1b';
        if (seq[2] == '~') {
          switch (seq[1]) {
            case '5': return PAGE_UP;
            case '6': return PAGE_DOWN;
          }
        }
      } else {
        switch (seq[1]) {
          case 'A': return ARROW_UP;
          case 'B': return ARROW_DOWN;
          case 'C': return ARROW_RIGHT;
          case 'D': return ARROW_LEFT;
        }
      }
    }
    return '\x1b';
  } else {
    return c;
  }
}
int getCursorPosition(int *rows, int *cols) { … }
int getWindowSize(int *rows, int *cols) { … }
/*** append buffer ***/
/*** output ***/
/*** input ***/
/*** init ***/
 ```

现在您了解了为什么我们声明`seq`能够存储`3个字节`。如果`[`后面的字节是数字，我们读取另一个字节，期望它是`〜`。然后，我们测试数字字节以查看它是`5`还是`6`。

让我们让Page Up和Page Down做点什么。目前，我们将让他们将光标移动到屏幕顶部或屏幕底部。 

```c
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
/*** append buffer ***/
/*** output ***/
/*** input ***/
void editorMoveCursor(int key) { … }
void editorProcessKeypress() {
  int c = editorReadKey();
  switch (c) {
    case CTRL_KEY('q'):
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case PAGE_UP:
    case PAGE_DOWN:
      {
        int times = E.screenrows;
        while (times--)
          editorMoveCursor(c == PAGE_UP ? ARROW_UP : ARROW_DOWN);
      }
      break;
    case ARROW_UP:
    case ARROW_DOWN:
    case ARROW_LEFT:
    case ARROW_RIGHT:
      editorMoveCursor(c);
      break;
  }
}
/*** init ***/
```

我们使用一对大括号创建一个代码块，以便我们可以声明`times`变量。（您不能在switch语句中直接声明变量。）我们模拟用户按下↑或↓键足够多的时间以移至屏幕顶部或底部。当我们实现滚动时，以这种方式实现Page Up和Page Down将使我们以后变得更加容易。 

 如果您使用带Fn键的笔记本电脑，则可以按Fn +↑和Fn +↓来模拟按Page Up和Page Down键。 

### `Home`和`END`键

现在，实现Home和End键。与先前的键一样，这些键也发送转义序列。与以前的键不同，这些键可以发送许多不同的转义序列，具体取决于您的操作系统或终端仿真器。

 Home键可以作为`<esc> [1〜`，`<esc> [7〜`，`<esc> [H`或`<esc> OH]`发送。类似地，End键可以作为`<esc> [4〜`，`<esc> [8〜`，`<esc> [F`或`<esc> OF]`发送。让我们处理所有这些情况。  

```c
/*** includes ***/
/*** defines ***/
#define KILO_VERSION "0.0.1"
#define CTRL_KEY(k) ((k) & 0x1f)
enum editorKey {
  ARROW_LEFT = 1000,
  ARROW_RIGHT,
  ARROW_UP,
  ARROW_DOWN,
  HOME_KEY,
  END_KEY,
  PAGE_UP,
  PAGE_DOWN
};
/*** data ***/
/*** terminal ***/
void die(const char *s) { … }
void disableRawMode() { … }
void enableRawMode() { … }
int editorReadKey() {
  int nread;
  char c;
  while ((nread = read(STDIN_FILENO, &c, 1)) != 1) {
    if (nread == -1 && errno != EAGAIN) die("read");
  }
  if (c == '\x1b') {
    char seq[3];
    if (read(STDIN_FILENO, &seq[0], 1) != 1) return '\x1b';
    if (read(STDIN_FILENO, &seq[1], 1) != 1) return '\x1b';
    if (seq[0] == '[') {
      if (seq[1] >= '0' && seq[1] <= '9') {
        if (read(STDIN_FILENO, &seq[2], 1) != 1) return '\x1b';
        if (seq[2] == '~') {
          switch (seq[1]) {
            case '1': return HOME_KEY;
            case '4': return END_KEY;
            case '5': return PAGE_UP;
            case '6': return PAGE_DOWN;
            case '7': return HOME_KEY;
            case '8': return END_KEY;
          }
        }
      } else {
        switch (seq[1]) {
          case 'A': return ARROW_UP;
          case 'B': return ARROW_DOWN;
          case 'C': return ARROW_RIGHT;
          case 'D': return ARROW_LEFT;
          case 'H': return HOME_KEY;
          case 'F': return END_KEY;
        }
      }
    } else if (seq[0] == 'O') {
      switch (seq[1]) {
        case 'H': return HOME_KEY;
        case 'F': return END_KEY;
      }
    }
    return '\x1b';
  } else {
    return c;
  }
}
int getCursorPosition(int *rows, int *cols) { … }
int getWindowSize(int *rows, int *cols) { … }
/*** append buffer ***/
/*** output ***/
/*** input ***/
/*** init ***/
```

 现在，让Home和End做些事情。现在，让他们将光标移动到屏幕的左边缘或右边缘。 

```
/*** includes ***/
/*** defines ***/
/*** data ***/
/*** terminal ***/
/*** append buffer ***/
/*** output ***/
/*** input ***/
void editorMoveCursor(int key) { … }
void editorProcessKeypress() {
  int c = editorReadKey();
  switch (c) {
    case CTRL_KEY('q'):
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case HOME_KEY:
      E.cx = 0;
      break;
    case END_KEY:
      E.cx = E.screencols - 1;
      break;
    case PAGE_UP:
    case PAGE_DOWN:
      {
        int times = E.screenrows;
        while (times--)
          editorMoveCursor(c == PAGE_UP ? ARROW_UP : ARROW_DOWN);
      }
      break;
    case ARROW_UP:
    case ARROW_DOWN:
    case ARROW_LEFT:
    case ARROW_RIGHT:
      editorMoveCursor(c);
      break;
  }
}
/*** init ***/
```

