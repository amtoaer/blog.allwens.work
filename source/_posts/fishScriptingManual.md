---
title: fish shell 脚本编写指南
date: 2022-05-06 12:33:21
tags: ['fish','linux']
---

> 该文章翻译自[FISH SHELL SCRIPTING MANUAL](https://developerlife.com/2021/01/19/fish-scripting-manual/)，因本人才疏学浅，难免有错误/不通顺的地方，还望读者在评论区不吝赐教。

通过例子学习如何编写 fish shell 脚本。

## 脚本顶部的 shebang 行

为了在终端中运行 fish 脚本，你需要做如下两件事：

1. 将以下 shebang 行添加到脚本顶部：`#!/usr/bin/env fish`。
2. 使用以下命令将文件标记为可执行：`chmod +x <你的 fish 脚本文件名>`。

<!--more-->

## 如何设置变量

注意，在 fish 中可以赋给变量的所有类型的值都是字符串，没有布尔值、整数或浮点数等概念。下面是一个给变量赋值的简单示例。这里有关于此的[更多信息](https://stackoverflow.com/questions/47761491/fish-shell-flip-a-boolean-value/47762934#47762934)。

```sh
set MY_VAR "some value"
```

你可以做的最有用的事情之一就是把 shell 中执行命令的输出存储到一个变量中。在你测试某个程序或命令是否返回了一些需要你执行其它命令的值（使用字符串比较、 if 语句和 switch 语句）时，这是很有用的。下面是一些这样做的例子。

```sh
set CONFIG_FILE_DIFF_OUTPUT (diff ~/Downloads/config.fish ~/.config/fish/config.fish)
set GIT_STATUS_OUTPUT (git status --porcelain)
```

### 变量作用域： local,global,global-export

有时你需要把变量导出到子进程，有时你需要把变量导出到全局作用域。还有一些时候，你希望把变量限制在正在编写的函数的局部作用域内。fish 文档中的 [`set`](https://fishshell.com/docs/current/cmds/set.html)函数部分有关于此的更多信息。

+ 把变量限制在函数的局部作用域内（即使有同名的全局变量）应该使用`set -l`。这种类型的变量不在整个 fish shell 可用。一个例子是仅用于保存函数范围内某个值的局部变量，如`set -l fname (realpath)`。
+ 使用`set -x`导出变量（仅在当前 fish shell 可用）。例如在一个运行在`crontab` headless 环境的 fish 函数中为 X11 会话设置`DISPLAY`环境变量。
+ 使用`set -gx`全局导出变量（可用于操作系统的任何程序，而不仅仅是当前运行的 fish shell 进程）。例如为本机上运行的所有程序设置`JAVA_HOME`环境变量。

### 列表

以下是一个在变量中追加值的例子。默认情况下 fish 变量是列表。

```sh
set MY_VAR $MY_VAR "another value"
```

这是创建列表的方法。

```sh
set MY_LIST "value1" "value2" "value3"
```

### 存储运行命令的返回值

下面是一个将命令执行返回的值存储到变量中的示例。

```sh
set OUR_VAR (math 1+2)
set OUR_VAR (date +%s)
set OUR_VAR (math $OUR_VAR / 60)
```

因为所有的 fish 变量都是列表，你可以使用 `[n]`运算符访问单个元素，**其中`n=1`表示第一个元素（而不是0）**。下面是一个例子。负数表示从末尾访问元素。

```sh
set LIST one two three
echo $LIST[1]  # one
echo $LIST[2]  # two
echo $LIST[3]  # three
echo $LIST[-1] # 和上一行等同
```

### 范围

你可以使用变量/列表的范围，接着上面的例子。

```sh
set LIST one two three
echo $LIST[1..2]  # one two
echo $LIST[2..3]  # two three
echo $LIST[-1..2] # three two
```

+ [这是列表的文档](https://fishshell.com/docs/current/tutorial.html#lists)

## 如何编写 for 循环

因为变量默认包含列表，所以很容易遍历他们。这是一个例子：

```sh
set FOLDERS bin
set FOLDERS $FOLDERS .atom
set FOLDERS $FOLDERS "my foldername"
for FOLDER in $FOLDERS
  echo "item: $FOLDER"
end
```

你也可以把`set`命令放在同一行以简化上述代码，就像这样：

```sh
set FOLDERS bin .atom "my foldername"
for FOLDER in $FOLDERS
  echo "item: $FOLDER"
end
```

还可以把整个 for 语句放到单行，像这样：

```sh
set FOLDERS bin .atom "my foldername"
for FOLDER in $FOLDERS ; echo "item: $FOLDER" ; end
```

## 如何编写 if 语句

编写 if 语句的关键是使用`test`命令对某个表达式求布尔值。可以是字符串比较，甚至是测试文件或文件夹的存在。下面是一些例子。你还可以使用`not`运算符作为`test`的前缀，以检查逆条件。

### 常用的条件

检查数组的大小。`$argv`包含从命令行传递给脚本的参数列表。

```sh
if test (count $argv) -lt 2
  echo "Usage: my-script <arg1> <arg2>"
  echo "Eg: <arg1> can be 'foo', <arg2> can be 'bar'"
else
  echo "👋 Do something with $arg1 $arg2"
end
```

变量的字符串比较。

```sh
if test $hostname = "mymachine"
  echo "hostname is mymachine"
end
```

检查文件是否存在：

```sh
if test -e "somefile"
  echo "somefile exists"
end
```

检查文件夹是否存在：

```sh
if test -d "somefolder"
  echo "somefolder exists"
end
```

检查文件通配符是否存在与文件和文件夹的检查略有不同。原因在于 fish 处理通配符的方式——在对它们执行任何命令之前，fish 会首先将其展开。

```sh
set -l files ~/Downloads/*.mp4 # 这个通配符表达式被展开，包含了实际的文件
if test (count $files) -gt 0
  mv ~/Downloads/*.mp4 ~/Videos/
  echo "📹 Moved '$files' to ~/Videos/"
else
  echo "⛔ No mp4 files found in Downloads"
end
```

一个在上述示例中使用`not`运算符的示例：

```sh
if not test -d "somefolder"
  echo "somefolder does not exist"
end
```

### 程序，脚本或函数的退出码

使用退出码的想法是，你的函数或者整个 fish 脚本可以被能够理解退出码的其他程序使用。换句话说，可能会有 if 语句使用退出码来确定某个条件。这是与其它命令行程序一起使用的一个非常常见的模式。退出码不同于函数的[返回值](#从函数中返回值)。

下面是一个使用`git`命令的退出码的示例：

```sh
if (git pull -f --rebase)
  echo "git pull with rebase worked without any issues"
else
  echo "Something went wrong that requires manual intervention, like a merge conflict"
end
```

一个测试命令执行是否没有错误的示例：

```sh
if sudo umount /media/user/mountpoint
  echo "Successfully unmounted /media/user/mountpoint"
end
```

你还可以检查`$status`变量的值。fish 会在执行命令后将返回值存储在这个变量中。这儿有关于此的[更多信息](https://fishshell.com/docs/2.3/faq.html)。

当你写函数时，可以使用以下关键字来退出函数或循环：`return`。`return`后可能会跟随一个数字，它的意思是：

1. `return`或`return 0` - 表示函数正常退出。
2. `return 1`或者其他大于0的数字 - 表示函数出了些问题。

你可以使用`exit`退出 fish shell 本身。整数退出码的含义与上述相同。

### set -q 和 test -z 的不同

在 if 语句中使用`set -q`和`test -z`检查变量是否为空时，有一些细微的区别。

1. 在使用`test -z`时，请确保使用引号包裹变量，因为如果变量不在引号中，该命令可能在某些边缘情况下出错。
2. 不过，你可以使用`set -q`来测试是否已经设置了变量，而无需将其括在引号中。

下面是示例：

```sh
set GIT_STATUS (git status --porcelain)
if set -q $GIT_STATUS ; echo "No changes in repo" ; end
if test -z "$GIT_STATUS" ; echo "No changes in repo" ; end
```

### 带有 and，or 运算符的多条件判断

如果想把多个条件组合到单个语句中，可以使用`or`和`and`运算符。如果想检查条件的逆，你可以使用`!`。下面是一个函数的示例，该函数用于检查命令行传递的两个参数。这是我们描述的逻辑：

1. 如果两个参数都缺失，应该向命令行打印帮助信息，然后提前`return`。
2. 如果其中一个参数缺失，则显示一个提示，说明其中一个参数缺失，然后提前`return`。

```sh
function requires-two-arguments
  # 没有参数传入
  if set -q "$argv"
    echo "Usage: requires-two-arguments arg1 arg2"
    return 1
  end
  # 只传入一个参数
  if test -z "$argv[1]"; or test -z "$argv[2]"
    echo "arg1 or arg2 can not be empty"
    return 1
  end
  echo "Thank you, got 1) $argv[1] and 2) $argv[2]"
end
```

下面是一些代码的注释：

1. [`set -q $variable`](https://fishshell.com/docs/current/cmds/set.html)函数做了什么？当`$variable`是空时，它返回 true。

2. 如果你想使用`test`函数替换掉`set -q`来判断一个变量是否存在，可以使用：

   + `if test -z "$variable"`
   + `if test ! -n "$variable"`或`if not test -n "$variable"`

3. 如果你想要把上面的`or`检查替换为`test`，它看起来会像：`if test -z "$argv[1]"; or test -z "$argv[2]"`。

   > 译者注：上面的代码本来就是`test -z`，这句话有点画蛇添足了。
   >
   > 推测作者的代码原文可能是`set -q "$argv[1]"; or set -q "$argv[2]"`，所以才会有这种表述。

4. 当你使用`or`，`and`运算符时，你必须使用`;`来结束条件表达式 。

5. 确保把变量括在空引号中。如果变量中包含一个空字符串，那么如果没有这些引号，语句将导致错误。

这是另一个用于测试`$variable`是否为空的例子：

```sh
if test -z "$variable" ; echo "empty" ; else ; echo "non-empty" ; end
```

这是另一个用于测试`$variable`是否包含字符串的例子：

```sh
if test -n "$variable" ; echo "non-empty" ; else ; echo "empty" ; end
```

### 另一个常见运算符： not

下面是一个使用`not`运算符测试字符串是否包含字符串片段的示例：

```sh
if not string match -q "*md" $argv[1]
  echo "The argument passed does not end in md"
else
  echo "The argument passed ends in md"
end
```

### 参考资料

1. [test command](https://fishshell.com/docs/current/cmds/test.html)
2. [set command](https://fishshell.com/docs/current/cmds/set.html)
3. [if command](https://fishshell.com/docs/current/cmds/if.html)
4. [stackoverflow answer: how to check if fish variable is empty](https://stackoverflow.com/questions/47743015/fish-shell-how-to-check-if-a-variable-is-set-empty)
5. [stackoverflow answer: how to put multiple conditions in fish if statement](https://stackoverflow.com/questions/17900078/in-fish-shell-how-can-i-put-two-conditions-in-an-if-statement)

## 如何使用分隔符拆分字符串

在某些情况下，您希望获取命令的输出（一个字符串），然后用某个分隔符将其分割，以便只使用输出字符串的一部分。例如获取给定文件的 SHA 校验和。命令`shasum <filename>`产生类似`df..d8 <filename>`的输出。假设我们只想要这个字符串的第一部分（SHA），已知分隔符是两个空格字符，我们可以执行以下操作来获取校验和部分，并将其存储在`$checksum`中。这里有关于`string split`命令的[更多信息](https://fishshell.com/docs/current/cmds/string-split.html)。

```sh
set CHECKSUM_ARRAY_STRING (shasum $FILENAME)
set CHECKSUM_ARRAY (string split "  " $SOURCE_CHECKSUM_ARRAY)
set CHECKSUM $CHECKSUM_ARRAY[1]
```

## 如何执行字符串比较

为了测试字符串中的子串匹配，你可以使用`string match`命令。这里有关于该命令的更多信息：

1. [string match 的官方文档](https://fishshell.com/docs/current/cmds/string-match.html)
2. [关于如何使用它的 Stackoverflow 回答](https://unix.stackexchange.com/a/504931/302646)

下面是一个实际应用的例子。注意，使用`-q`或`--quiet`时，如果匹配条件满足（成功），它不会打印字符串的输出。

```sh
if string match -q "*myname*" $hostname
  echo "$hostname contains myname"
else
  echo "$hostname does not contain myname"
end
```

下面是一个字符串精确匹配的示例：

```sh
if test $hostname = "machine-name"
  echo "Exact match"
else
  echo "Not exact match"
end
```

一个测试字符串是否为空的示例：

```sh
if set -q $my_variable
  echo "my_variable is empty"
end
```

下面是一个复杂的示例，测试是否安装了`ruby-dev`和`ruby-bundler`包。如果是，则运行`jekyll`；如果不是，则安装这些包。

```sh
# Return "true" if $packageName is installed, and "false" otherwise.
# Use it in an if statement like this:
#
# if string match -q "false" (isPackageInstalled my-package-name)
#   echo "my-package-name is not installed"
# else
#   echo "my-package-name is installed"
# end
function isPackageInstalled -a packageName
  set packageIsInstalled (dpkg -l "$packageName")
  if test -z "$packageIsInstalled"
    set packageIsInstalled false
  else
    set packageIsInstalled true
  end
  echo $packageIsInstalled
end

# More info to find if a package is installed: https://askubuntu.com/a/823630/872482
if test (uname) = "Linux"

  echo "🐒isPackageInstalled does-not-exist:" (isPackageInstalled does-not-exist)

  if string match -q "false" (isPackageInstalled ruby-dev) ;
    or string match -q "false" (isPackageInstalled ruby-bundler)
    # Install ruby
    echo "ruby-bundler or ruby-dev are not installed; installing now..."
    echo sudo apt install -y ruby-bundler ruby-dev
  else
    bundle install
    bundle update
    bundle exec jekyll serve
  end

end
```

## 如何为字符串编写 switch 语句

为了为字符串创建 switch 语句，这里也使用了`test`命令(就像用于[ if 语句]()一样)。`case`语句需要匹配子字符串，可以使用通配符和想要匹配的子字符串的组合来表示子字符串。这是一个示例。

```sh
switch $hostname
case "*substring1*"
  echo "Matches $hostname containing substring1"
case "*substring2*"
  echo "Matches $hostname containing substring2"
end
```

你也可以把这些和 if 语句混合使用，最终看起来会像这样：

```sh
if test (uname) = "Darwin"
  echo "Machine is running macOS"
  switch $hostname
  case "*MacBook-Pro*"
    echo "hostname has MacBook-Pro in it"
  case "*MacBook-Air*"
    echo "hostname has MacBook-Air in it"
  end
else
  echo "Machine is not running macOS"
end
```

## 如何执行字符串

执行脚本中生成的字符串最安全的方法是使用以下模式。

```sh
echo "ls \
  -la" | sh
```

这不仅使调试更容易，还在使用`\`进行多行换行时避免了奇怪的错误。

## 如何编写函数

一个 fish 函数只是一个可选地接受参数的命令列表。这些参数作为列表传入（因为 fish 的所有变量都是列表）。

这是一个例子：

```sh
function say_hi
  echo "Hi $argv"
end
say_hi
say_hi everbody!
say_hi you and you and you
```

写完一个函数后，你可以通过使用`type`查看它是什么。例如：`type say_hi`将展示你刚刚创建的函数。

+ [这是函数的文档](https://fishshell.com/docs/current/tutorial.html#functions)

### 向函数传递参数

除了使用`$argv`找出传递给函数的参数外，你还可以提供一个函数所期望的具名参数列表。这是[官方文档中的更多信息](https://fishshell.com/docs/current/cmds/function.html)。

需要记住的一些关键事项：

1. 参数名不能含有`-`字符，使用`_`替代。
2. 不要使用`(`和`)`向函数传递参数，只需要在带空格的单行中传递参数即可。

以下是一个示例：

```sh
function testFunction -a param1 param2
  echo "arg1 = $param1"
  echo "arg2 = $param2"
end
testFunction A B
```

下面是另一个测试传递给函数的参数是否存在的示例：

```sh
# Note parameter names can't have dashes in them, only underscores.
function my-function -a extension search_term
  if test (count $argv) -lt 2
    echo "Usage: my-function <extension> <search_term>"
    echo "Eg: <extension> can be 'fish', <search_term> can be 'test'"
  else
    echo "✋ Do something with $extension $search_term"
  end
end
```

### 从函数中返回值

你可能需要从函数返回值（通常只是一个字符串），还可以返回许多由新行分割的字符串。无论如何，实现这一目标的机制是相同的。只需要使用`echo`将返回值转储到 stdout 即可。

这是一个示例：

```sh
function getSHAForFilePath -a filepath
  set NULL_VALUE ""
  # No $filepath provided, or $filepath does not exist -> early return w/ $NULL_VALUE.
  if set -q $filepath; or not test -e $filepath
    echo $NULL_VALUE
    return 0
  else
    set SHASUM_ARRAY_STRING (shasum $filepath)
    set SHASUM_ARRAY (string split "  " $SHASUM_ARRAY_STRING)
    echo $SHASUM_ARRAY[1]
  end
end

function testTheFunction
  echo (getSHAForFilePath ~/local-backup-restore/does-not-exist.fish)
  echo (getSHAForFilePath)
  set mySha (getSHAForFilePath ~/local-backup-restore/test.fish)
  echo $mySha
end

testTheFunction
```

## 如何处理依赖项的文件和文件夹路径

随着脚本变得更加复杂，您可能需要处理加载多个脚本的问题。在这种情况下，您可以使用`source my-script.sh`从当前脚本导入其它脚本。然而，fish 会在当前目录，即你开始执行脚本的目录寻找`my-script.sh`文件，而该目录可能与你需要加载此依赖项的位置不匹配。如果你的主脚本在`$PATH`中而依赖项不在，就会发生这种情况。在这种情况下，可以在主脚本中执行以下操作：

```sh
set MY_FOLDER_PATH (dirname (status --current-filename))
source $MY_FOLDER_PATH/my-script.fish
```

这段代码实际做的是获取主脚本运行的文件夹，并将其存储在`MY_FOLDER_PATH`中，然后就可以使用`source`命令加载任何依赖项了。这种方法有一个限制，即在`MY_FOLDER_PATH`中存储的是相对于主脚本执行位置的路径。这是一个你可能不关心的微妙细节，除非你需要一个绝对路径名。在这种情况下，你可以执行以下操作：

```sh
set MY_FOLDER_PATH (realpath (dirname (status --current-filename)))
source $MY_FOLDER_PATH/my-script.fish
```

使用[`realpath`](https://man7.org/linux/man-pages/man1/realpath.1.html)可以提供文件夹的绝对路径，以便在需要此功能的情况下使用。

## 如何将多行字符串写入文件

在许多情况下，你需要在脚本中将字符串和多行字符串写入到新的或者现有的文件。

以下是一个向文件中写入单行字符串的例子：

```sh
# echo "echo 'ClientAliveInterval 60' >> recurring-tasks.log" | xargs -I% sudo sh -c %
set linesToAdd "TCPKeepAlive yes" "ClientAliveInterval 60" "ClientAliveCountMax 120"
for line in $linesToAdd
  set command "echo '$line' >> /etc/ssh/sshd_config"
  executeString "$command | xargs -I% sudo sh -c %"
end
```

下面是一个向文件写入多行字符串的例子：

```sh
# More info on writing multiline strings: https://stackoverflow.com/a/35628657/2085356
function _workflowWriteEmptyMarkdownContentToFile --argument datestr filename
  echo > $filename "\
---
Title: About $filename
Date: $datestr
---
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Your heading
"
end
```

## 如何创建彩色的 echo 输出

[`set_color`](https://fishshell.com/docs/current/cmds/set_color.html)函数允许 fish 对使用`echo`打印到`stdout`的文本内容进行着色和格式化。这在创建需要不同前景、背景颜色以及粗体、斜体或下划线输出的文本输出时非常有用。有很多方法来使用该命令，以下是两个例子（`echo`语句内联，以及单独使用）：

```sh
function myFunction
  if test (count $argv) -lt 2
    set -l currentFunctionName (status function)
    echo "Usage: "(set_color -o -u)"$currentFunctionName"(set_color normal)\
      (set_color blue)" <arg1> "\
      (set_color yellow)"<arg2>"(set_color normal)
    set_color blue
    echo "- <arg1>: Something about arg1."
    set_color yellow
    echo "- <arg2>: Something about arg2"
    set_color normal
    return 1
  end
end
```

注意：

1. 必须调用`set_color normal`来重置以前语句中设置的格式化选项。
2. `set_color -u`表示下划线，`set_color -o`表示粗体。

## 如何获取用户输入

在某些情况下，你需要在执行一些可能具有破坏性的操作之前要求用户进行确认，或者可能需要用户输入函数的某些参数（该参数不是通过命令行传递的）。在这些情况下，可以通过使用[`read`](https://fishshell.com/docs/current/cmds/read.html)函数读取`stdin`以获取用户输入。

下面的函数简单地为 'Y'/'y' 返回 0，为 'N'/'n' 返回 1。

```sh
# More info on prompting a user for confirmation using fish read function: https://stackoverflow.com/a/16673745/2085356
# More info about fish `read` function: https://fishshell.com/docs/current/cmds/read.html
function _promptUserForConfirmation -a message
  if not test -z "$message"
    echo (set_color brmagenta)"🤔 $message?"
  end

  while true
    # read -l -P '🔴 Do you want to continue? [y/N] ' confirm
    read -l -p "set_color brcyan; echo '🔴 Do you want to continue? [y/N] ' ; set_color normal; echo '> '" confirm
    switch $confirm
      case Y y
        return 0
      case '' N n
        return 1
    end
  end
end
```

下面是使用`_promptUserForConfirmation`函数的一个示例：

```sh
if _promptUserForConfirmation "Delete branch $featureBranchName"
  git branch -D $featureBranchName
  echo "👍 Successfully deleted $featureBranchName"
else
  echo "⛔ Did not delete $featureBranchName"
end
```

## 如何使用 sed

这对删除不需要的文件片段非常有用。特别是在使用`xargs`对`find`结果进行管道处理时。

以下是一个从每个文件的开头删除 './' 的示例：

```sh
echo "./.Android" | sed 's/.\///g'
```

一个一起使用`sed`，`find`和`xargs`的更复杂的例子：

```sh
set folder .Android*
find ~ -maxdepth 1 -name $folder | sed 's/.\///g' | \
  xargs -I % echo "cleaned up name: %"
```

## 如何使用 xargs

这对于将某些命令的输出作为更多命令的参数很有用。

这是一个简单的例子：`ls | xargs echo "folders: "`。

1. 产生的输出是：`folders: idea-http-proxy-settings images tmp`
2. 注意参数在输出中是如何连接的

下面是一个稍微不同的例子，使用`-I %`可以将参数放在任何地方（而不仅仅是放在末尾）。

```sh
ls | xargs -I % echo "folder: %"
```

产生如下输出：

```sh
folder: idea-http-proxy-settings
folder: images
folder: tmp
```

注意，每个参数都在单独的一行中。

## 如何使用 cut 切分字符串

假设你有一个字符串`"token1:token2"`，你想切分这个字符串并只保留它的第一部分，这可以使用下述的`cut`命令完成。

```sh
echo "token1:token2" | cut -d ':' -f 1
```

+ `-d ':'` - 将通过`:`分隔符分割字符串
+ `-f 1` - 保留 token 化的字符串中的第一个字段

下面是一个在`~/github/developerlife.com`中查找所有含有`fonts.googleapis`的 HTML 文件，然后使用`subl`打开的示例：

```sh
cd ~/github/developerlife.com
echo \
"find . -name '*html' | \
 xargs grep fonts.googleapis | \
 cut -d ':' -f 1 | \
 xargs subl" \
 | sh
```

## 如何计算脚本运行所需的时间

```sh
function timed -d Pass the program or function that you want to execute as an argument
  set START_TS (date +%s)

  # This is where your code would go.
  $argv

  sleep 5

  set END_TS (date +%s)
  set RUNTIME (math $END_TS - $START_TS)
  set RUNTIME (math $RUNTIME / 60)
  echo "⏲ Total runtime: $RUNTIME min ⏲"
end
```

