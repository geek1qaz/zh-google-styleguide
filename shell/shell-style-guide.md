# Shell 风格指南

## 扉页

* 版本：2.02
* 作者：Googlers
* 翻译：[GitHub@geek1qaz](https://github.com/geek1qaz/) v1.0
* 项目：[Google Style Guides](https://github.com/google/styleguide)

## 背景

### 使用哪一种 Shell

Bash 是唯一被允许执行的 shell 脚本语言。

可执行文件必须以 `#!/bin/bash` 和尽可能少的标识符（flag）开始。请使用 `set` 来设置 shell 的选项，使得用 `bash script_name` 调用您的脚本时不会破坏其功能。

限制所有的可执行 shell 脚本为 bash 使得我们安装在所有计算机中的 shell 语言保持一致性。

对此唯一的例外是您被您编码的任何东西所强迫。其中一个例子是 Solaris SVR4 包，它需要普通的 Bourne shell 来运行任何脚本。

### 什么时候使用 Shell

Shell 应该只用于小型实用程序或简单的包装脚本。

虽然 shell 脚本不是一种开发语言，但在 Google 它被用于编写各种实用程序脚本。本风格指南更多的是对其使用的认可，而不是建议将其用于广泛部署。

以下是一些指导方针：

* 如果您主要是调用其他实用程序并且执行相对较少的数据操作，那么 shell 是该任务可接受的选择。
* 如果您对性能要求很高，那么请选择其他工具，而不是使用 shell。
* 如果您正在编写的脚本超过 100 行，或者使用非直接的控制流逻辑，那么您现在应该用更结构化的语言重写它。请记住，脚本会增长。尽早重写您的脚本，以避免在以后进行更耗时的重写。
* 在评估代码的复杂性（例如，决定是否切换语言）时，请从维护者的角度出发而非脚本作者的角度出发。

## Shell 文件和解释器调用

### 文件扩展名

可执行文件应该没有扩展名（强烈推荐）或者使用 `.sh` 扩展名。库文件必须使用 `.sh` 扩展名，并且应该是不可执行的。

执行程序时没有必要知道程序是用什么语言编写的，而且 shell 脚本也不要求有扩展名，因此我们更喜欢可执行文件没有扩展名。

然而，对于库文件来说，知道它是什么语言很重要，有时需要在不同的语言中有类似的库。 这允许具有相同用途但不同语言的库文件具有相同的名称，除了特定于语言的后缀。

### SUID/SGID

SUID 和 SGID 在 shell 脚本中是被禁止的。

Shell 存在太多的安全问题，以致于如果允许 SUID/SGID 会使得它几乎不可能足够安全。虽然 bash 使得运行 SUID 非常困难， 但在某些平台上它仍然是可能的，这就是为什么我们明确提出要禁止它。

如果您需要较高权限的访问请使用 `sudo`。

## 环境

### STDOUT vs STDERR

所有错误信息都应该输出到 `STDERR`。

这样就更容易将正常的执行状态与实际问题区分开来。

建议使用一个函数来打印错误消息及其他状态信息。

`````bash
err() {
  echo "[$(date +'%Y-%m-%dT%H:%M:%S:%z')]: $*" >&2
}

if ! do_something; then
  err "Unable to do_something"
  exit 1
fi
`````

## 注释

### 文件头

每个文件的开头需要有描述脚本内容的信息。

每个文件都必须有一个顶级注释，包含对其内容的简要概述。版权声明和作者信息是可选的。

例如：

````bash
#!/bin/bash
#
# Perform hot backups of Oracle databases.
````

### 函数注释

任何内容晦涩且长度不短的函数都必须注释。所有库函数都必须注释，无论其长度或复杂性如何。

其他人通过阅读注释（和帮助信息，如果有的话）就能够学会如何使用您的程序或使用库中的函数，而无需阅读代码。

所有函数注释都应该使用以下语句描述预期的 API 行为：

* 函数的功能描述
* 全局变量 (Globals)：使用和修改的全局变量列表
* 参数 (Arguments)：函数接收的参数
* 输出 (Outputs)：输出到 STDOUT 或 STDERR 的内容
* 返回 (Returns)：除最后执行的命令的默认退出状态以外的返回值

例如：

````bash
#######################################
# Cleanup files from the backup directory.
# Globals:
#   BACKUP_DIR
#   ORACLE_SID
# Arguments:
#   None
#######################################
function cleanup() {
  ...
}

#######################################
# Get configuration directory.
# Globals:
#   SOMEDIR
# Arguments:
#   None
# Outputs:
#   Writes location to stdout
#######################################
function get_dir() {
  echo "${SOMEDIR}"
}

#######################################
# Delete a file in a sophisticated manner.
# Arguments:
#   File to delete, a path.
# Returns:
#   0 if thing was deleted, non-zero on error.
#######################################
function del_thing() {
  rm "$1"
}
````

### 实现部分的注释

注释代码中棘手的、不明显的、有趣的或重要的部分。

这部分遵循谷歌代码注释的通用做法。不要注释所有代码。如果有一个复杂的算法，或者你正在做一些不寻常的事情，请添加一个简短的注释。

### TODO 注释

对于临时的、短期的解决方案，或者足够好但不够完美的代码，使用 `TODO` 注释。

这与 C++ 指南中的约定相一致。

`TODO`s 应该包含全大写形式的字符串 `TODO`，后面跟着姓名、电子邮件地址、bug ID 或者个人或问题的其他标识信息，并提供 `TODO` 所引用问题的最佳上下文。冒号是可选的。主要目的是有一个一致的 `TODO`，可以通过搜索来查找如何根据请求获得更多详细信息。`TODO` 不是指被引用的人会解决问题的承诺。因此，当您创建 `TODO` 并标注姓名时，给出的几乎总是您的名字。

例如：

````bash
# TODO(mrmonkey): Handle the unlikely edge cases (bug ####)
# TODO(kl@gmail.com): Use a "*" here for concatenation operator.
# TODO(Zeke) change this to use relations.
# TODO(bug 12345): remove the "Last visitors" feature.
````

如果您的 `TODO` 是“在未来的某天做某事”的形式，请确保您包含一个非常具体的日期（“在 2005 年 11月之前修复”）或一个非常具体的事件（“当所有客户端都可以处理 XML 响应时删除此代码”）。

## 格式

虽然您应该对正在修改的文件遵循既有的样式，但对于任何新代码，以下是必需的。

### 缩进

缩进 2 个空格。不要使用制表符。

在代码块之间请使用空行以提高可读性。缩进是两个空格。无论如何都不要使用制表符。对于现有的文件，保持现有的缩进格式。

### 行的长度和长字符串

行的最大长度为 80 个字符。

如果必须编写超过 80 个字符的字符串，应该尽可能使用 here document 或者嵌入的换行符。长度超过80个字符的文字串且不能被合理地分割，这是正常的。但强烈建议找到一种方法使其变短。

````bash
# 使用 'here document'
cat <<END
I am an exceptionally long
string.
END

# 嵌入换行符也可以
long_string="I am an exceptionally
long string."
````

### 管道

如果一行容不下整个管道操作，那么请将整个管道操作分割成每行一个管段。

如果一行容得下整个管道操作，那么请将整个管道操作写在同一行。

否则，应该将整个管道操作分割成每行一个管段，管道操作的下一部分应该将管道符放在新行并且缩进 2 个空格。这适用于使用 `|` 组合的命令链，以及使用 `||` 和 `&&` 的逻辑运算链。

````bash
# All fits on one line
command1 | command2

# Long commands
command1 \
  | command2 \
  | command3 \
  | command4
````

### 循环

请将 `; do`、`; then` 和 `while`、`for`、`if` 放在同一行。

Shell 中的循环略有不同，但是我们遵循跟声明函数时的大括号相同的原则。也就是说，`; do`、`; then` 应该和 if/for/while 放在同一行。`else` 应该单独一行，结束语句应该单独一行并且跟开始语句垂直对齐。

例如：

```bash
# If inside a function, consider declaring the loop variable as
# a local to avoid it leaking into the global environment:
# local dir
for dir in "${dirs_to_cleanup[@]}"; do
  if [[ -d "${dir}/${ORACLE_SID}" ]]; then
    log_date "Cleaning up old files in ${dir}/${ORACLE_SID}"
    rm "${dir}/${ORACLE_SID}/"*
    if (( $? != 0 )); then
      error_message
    fi
  else
    mkdir -p "${dir}/${ORACLE_SID}"
    if (( $? != 0 )); then
      error_message
    fi
  fi
done
```

### Case 语句

* 将选项缩进 2 个空格。

* 单行选项需要在模式右括号之后和结束符 `;;` 之前各有一个空格。
* 长选项或多命令选项应该被拆分成多行，模式、操作和结束符 `;;` 在不同的行。

匹配表达式比 `case` 和 `esac` 缩进一级。多行操作要再缩进一级。一般来说，不需要引用匹配表达式。模式表达式的前面不应该有左括号。避免使用 `;&` 和 `;;&` 符号。

````bash
case "${expression}" in
  a)
    variable="…"
    some_command "${variable}" "${other_expr}" …
    ;;
  absolute)
    actions="relative"
    another_command "${actions}" "${other_expr}" …
    ;;
  *)
    error "Unexpected expression '${expression}'"
    ;;
esac
````

只要表达式保持可读，简单命令可以与模式和 `;;` 写在同一行。这通常适用于单字母选项的处理。当单行容不下操作时，请将模式单独放一行，然后是操作，最后结束符 `;;` 也单独一行。当与操作在同一行时，模式的右括号之后和结束符 `;;` 之前请使用一个空格分隔。

`````bash
verbose='false'
aflag=''
bflag=''
files=''
while getopts 'abf:v' flag; do
  case "${flag}" in
    a) aflag='true' ;;
    b) bflag='true' ;;
    f) files="${OPTARG}" ;;
    v) verbose='true' ;;
    *) error "Unexpected option ${flag}" ;;
  esac
done
`````

### 变量扩展

按照优先级顺序：与您发现的保持一致；引用您的变量；选择 `"${var}"` 而不是 `"$var"`。

这些是强烈推荐的指导方针，但不是强制规定。尽管这只是一种建议，不是强制性的，但这并不意味着我们应该轻视或草率对待它。

以下按照优先顺序列出：

* 与现存代码中您所发现的保持一致。

* 引用变量，请参阅下一小节。

* 除非绝对必要或者为了避免深深的困惑，否则不要用大括号将单个字符的 shell 特殊变量或位置变量括起来。

  推荐将其他所有变量用大括号括起来。

````bash
# Section of *recommended* cases.

# Preferred style for 'special' variables:
echo "Positional: $1" "$5" "$3"
echo "Specials: !=$!, -=$-, _=$_. ?=$?, #=$# *=$* @=$@ \$=$$ …"

# Braces necessary:
echo "many parameters: ${10}"

# Braces avoiding confusion:
# Output is "a0b0c0"
set -- a b c
echo "${1}0${2}0${3}0"

# Preferred style for other variables:
echo "PATH=${PATH}, PWD=${PWD}, mine=${some_var}"
while read -r f; do
  echo "file=${f}"
done < <(find /tmp)
````

````bash
# Section of *discouraged* cases

# Unquoted vars, unbraced vars, brace-delimited single letter
# shell specials.
echo a=$avar "b=$bvar" "PID=${$}" "${1}"

# Confusing use: this is expanded as "${1}0${2}0${3}0",
# not "${10}${20}${30}
set -- a b c
echo "$10$20$30"
````

注意：在 `${var}` 中使用大括号不是引用的形式。双引号也必须使用。

### 引用

* 除非需要谨慎地进行不带引用的扩展或者它是 shell 内部的整数（请参阅下一小节），否则总是引用包含变量、命令替换、空格或 shell 元字符的字符串。
* 使用数组安全引用元素列表，特别是命令行标识符。请参阅下面的数组。
* 可选（optionally）引用 shell 内部的、定义为整数的只读特殊变量：`$?`、`$#`、`$$` 、`$!`(man bash)。为了保持一致性，推荐引用“命名的（named）”内部整型变量，例如 PPID 等。
* 推荐使用引号括起来作为“单词（words）”的字符串（而不是命令选项或路径名称）。
* 永远不要引用整数字面值（literal integers）。
* 注意 `[[ ... ]]` 中模式匹配的引用规则。请参阅下面的 Test, `[ ... ]` 和 `[[ ... ]]` 小节。
* 除非您有使用 `$*` 的特殊理由，否则请使用 `"$@"`，例如简单地将参数附加到消息或日志中的字符串。

``````bash
# 'Single' quotes indicate that no substitution is desired.
# "Double" quotes indicate that substitution is required/tolerated.

# Simple examples

# "quote command substitutions"
# Note that quotes nested inside "$()" don't need escaping.
flag="$(some_command and its args "$@" 'quoted separately')"

# "quote variables"
echo "${flag}"

# Use arrays with quoted expansion for lists.
declare -a FLAGS
FLAGS=( --foo --bar='baz' )
readonly FLAGS
mybinary "${FLAGS[@]}"

# It's ok to not quote internal integer variables.
if (( $# > 3 )); then
  echo "ppid=${PPID}"
fi

# "never quote literal integers"
value=32
# "quote command substitutions", even when you expect integers
number="$(generate_number)"

# "prefer quoting words", not compulsory
readonly USE_INTEGER='true'

# "quote shell meta characters"
echo 'Hello stranger, and well met. Earn lots of $$$'
echo "Process $$: Done making \$\$\$."

# "command options or path names"
# ($1 is assumed to contain a value here)
grep -li Hugo /dev/null "$1"

# Less simple examples
# "quote variables, unless proven false": ccs might be empty
git send-email --to "${reviewers}" ${ccs:+"--cc" "${ccs}"}

# Positional parameter precautions: $1 might be unset
# Single quotes leave regex as-is.
grep -cP '([Ss]pecial|\|?characters*)$' ${1:+"$1"}

# For passing on arguments,
# "$@" is right almost every time, and
# $* is wrong almost every time:
#
# * $* and $@ will split on spaces, clobbering up arguments
#   that contain spaces and dropping empty strings;
# * "$@" will retain arguments as-is, so no args
#   provided will result in no args being passed on;
#   This is in most cases what you want to use for passing
#   on arguments.
# * "$*" expands to one argument, with all args joined
#   by (usually) spaces,
#   so no args provided will result in one empty string
#   being passed on.
# (Consult `man bash` for the nit-grits ;-)

(set -- 1 "2 two" "3 three tres"; echo $#; set -- "$*"; echo "$#, $@")
(set -- 1 "2 two" "3 three tres"; echo $#; set -- "$@"; echo "$#, $@")
``````

## 特性和漏洞

### ShellCheck

ShellCheck 项目为您的 shell 脚本识别常见的错误和警告。建议所有脚本都使用这个选项，无论大小。

### 命令替换

使用 `$(command)` 而不是反引号。

嵌套的反引号要求用 `\` 转义内部的反引号。而 `$(command)` 嵌套时格式不需要改变，并且更容易阅读。

例如：

````bash
# This is preferred:
var="$(command "$(command1)")"
````

````bash
# This is not:
var="`command \`command1\``"
````

### Test, `[ ... ]` 和 `[[ ... ]]`

首选 `[[ ... ]]`，而不是 `[ ... ]`、`test` 和 `/usr/bin/[`。

因为在 `[[` 和 `]]` 之间不会有路径名称扩展或单词分割发生，所以使用 `[[ ... ]]` 能够减少错误。而且 `[[ ... ]]` 允许正则表达式匹配，而 `[ ... ]` 不允许。

`````bash
# This ensures the string on the left is made up of characters in
# the alnum character class followed by the string name.
# Note that the RHS should not be quoted here.
if [[ "filename" =~ ^[[:alnum:]]+name ]]; then
  echo "Match"
fi

# This matches the exact pattern "f*" (Does not match in this case)
if [[ "filename" == "f*" ]]; then
  echo "Match"
fi
`````

````bash
# This gives a "too many arguments" error as f* is expanded to the
# contents of the current directory
if [ "filename" == f* ]; then
  echo "Match"
fi
````

具体细节，请参见 http://tiswww.case.edu/php/chet/bash/FAQ 的 E14。

### 字符串测试

尽可能使用引用，而不是填充字符。

Bash 足以在测试中处理空字符串。因此，请使用空（非空）字符串测试，而不是填充字符，使得代码更易于阅读。

````bash
# Do this:
if [[ "${my_var}" == "some_string" ]]; then
  do_something
fi

# -z (string length is zero) and -n (string length is not zero) are
# preferred over testing for an empty string
if [[ -z "${my_var}" ]]; then
  do_something
fi

# This is OK (ensure quotes on the empty side), but not preferred:
if [[ "${my_var}" == "" ]]; then
  do_something
fi
````

````bash
# Not this:
if [[ "${my_var}X" == "some_stringX" ]]; then
  do_something
fi
````

为避免对您测试的目的产生困惑，请明确使用 `-z` 或 `-n`。

```bash
# Use this
if [[ -n "${my_var}" ]]; then
  do_something
fi
```

````bash
# Instead of this
if [[ "${my_var}" ]]; then
  do_something
fi
````

为了清晰起见，使用 `==` 来表示相等，而不是 `=`，尽管两者都可以。前者鼓励使用 `[[`，后者可能会与赋值混淆。但是，在 `[[ ... ]]` 中使用 `<` 和 `>` 时要小心，因为它执行字典序比较。使用 `(( ... ))` 或 `-lt` 和 `-gt` 进行数值比较。

````bash
# Use this
if [[ "${my_var}" == "val" ]]; then
  do_something
fi

if (( my_var > 3 )); then
  do_something
fi

if [[ "${my_var}" -gt 3 ]]; then
  do_something
fi
````

````bash
# Instead of this
if [[ "${my_var}" = "val" ]]; then
  do_something
fi

# Probably unintended lexicographical comparison.
if [[ "${my_var}" > 3 ]]; then
  # True for 4, false for 22.
  do_something
fi
````

### 文件名的通配符扩展

在对文件名进行通配符扩展时，请使用明确的路径。

因为文件名可能以 `-` 开头，所以使用 `./*` 而不是 `*` 扩展通配符要安全得多。

````bash
# Here's the contents of the directory:
# -f  -r  somedir  somefile

# Incorrectly deletes almost everything in the directory by force
psa@bilby$ rm -v *
removed directory: `somedir'
removed `somefile'
````

```bash
# As opposed to:
psa@bilby$ rm -v ./*
removed `./-f'
removed `./-r'
rm: cannot remove `./somedir': Is a directory
removed `./somefile'
```

### Eval

应该避免使用 `eval`。

Eval 在用于变量赋值时会调整输入，并且能够设置变量，但无法检查这些变量是什么。

```bash
# What does this set?
# Did it succeed? In part or whole?
eval $(set_my_variables)

# What happens if one of the returned values has a space in it?
variable="$(eval some_function)"
```

### 数组

应该使用 Bash 数组来存储元素列表，以避免引用复杂性。这尤其适用于参数列表。数组不应用于促成更复杂的数据结构（请参阅上面什么时候使用 Shell）。

数组存储字符串的有序集合，可以安全地扩展为单个元素，用于命令或循环。

应该避免将一个字符串用于多个命令参数，因为这不可避免地会导致作者使用 `eval` 或试图在字符串中嵌套引号，这不会给出可靠或可读的结果，并且会导致不必要的复杂性。

````bash
# An array is assigned using parentheses, and can be appended to
# with +=( … ).
declare -a flags
flags=(--foo --bar='baz')
flags+=(--greeting="Hello ${name}")
mybinary "${flags[@]}"
````

````bash
# Don’t use strings for sequences.
flags='--foo --bar=baz'
flags+=' --greeting="Hello world"'  # This won’t work as intended.
mybinary ${flags}
````

````bash
# Command expansions return single strings, not arrays. Avoid
# unquoted expansion in array assignments because it won’t
# work correctly if the command output contains special
# characters or whitespace.

# This expands the listing output into a string, then does special keyword
# expansion, and then whitespace splitting.  Only then is it turned into a
# list of words.  The ls command may also change behavior based on the user's
# active environment!
declare -a files=($(ls /directory))

# The get_arguments writes everything to STDOUT, but then goes through the
# same expansion process above before turning into a list of arguments.
mybinary $(get_arguments)
````

#### 数组的优点

* 使用数组可以在不混淆引用语义的情况下使用列表。相反，不使用数组会导致在字符串中嵌套引用的错误尝试。
* 数组可以安全地存储任意字符串的序列/列表，包括包含空格的字符串。

#### 数组的缺点

使用数组可能会使脚本的复杂性增加。

#### 数组的抉择

数组应该被用于安全地创建和传递列表。特别是在构建一组命令参数时，请使用数组以避免混淆引用问题。使用带引号的扩展 - `"${array[@]}"` - 来访问数组。但是，如果需要更高级的数据操作，则应完全避免使用 shell 脚本； 看上面。

### 管道导向 while 循环

使用进程替换或内置的 readarray（bash4+），而不是管道导向 `while` 循环。管道会创建一个子 shell，因此管道内修改的任何变量都不会传递到父 shell。

管道导向 `while` 循环中隐含的子 shell 可能会引入难以追踪的细微错误。

```bash
last_line='NULL'
your_command | while read -r line; do
  if [[ -n "${line}" ]]; then
    last_line="${line}"
  fi
done

# This will always output 'NULL'!
echo "${last_line}"
```

使用进程替换也会创建一个子 shell。但是，它允许从子 shell 重定向到 `while` 循环，而不需要在子 shell 中放入 `while`（或任何其他命令）。

````bash
last_line='NULL'
while read line; do
  if [[ -n "${line}" ]]; then
    last_line="${line}"
  fi
done < <(your_command)

# This will output the last non-empty line from your_command
echo "${last_line}"
````

或者，使用内置的 `readarray` 将文件读入数组，然后循环遍历数组的内容。请注意（出于与上述相同的原因）您需要使用带有 `readarray` 而不是管道的进程替换，但其优点是循环的输入生成在它之前，而不是之后。

```bash
last_line='NULL'
readarray -t lines < <(your_command)
for line in "${lines[@]}"; do
  if [[ -n "${line}" ]]; then
    last_line="${line}"
  fi
done
echo "${last_line}"
```

> 注意：请谨慎使用 for 循环来迭代输出，例如 `for var in $(...)`，因为输出是按空格分割的，而不是按行。有时你能知道这是安全的，因为输出不包含任何意外的空格，但是在这不明显或不能提高可读性的地方（例如 `$(...)` 中的长命令），`while read` 循环或 `readarray` 通常更安全、更清晰。

### 算术

始终使用 `(( ... ))` 或 `$(( ... ))` 而不是 `let` 或 `$[ ... ]` 或 `expr`。

永远不要使用 `$[ ... ]` 语法、`expr` 命令或内置的 `let`。

`<` 和 `>` 在 `[[ ... ]]` 表达式中不执行数值比较（它们执行字典序比较；请参阅字符串测试）。作为首选，根本不要使用 `[[ ... ]]` 进行数值比较，而是使用 `(( ... ))`。

建议避免将 `$(( ... ))` 作为独立语句使用，否则要小心其表达式的值为零。

* 特别是在启用 `set -e` 的情况下。例如，执行 `set -e; i=0; (( i++ ))` 将导致 shell 退出。

```bash
# Simple calculation used as text - note the use of $(( … )) within
# a string.
echo "$(( 2 + 2 )) is 4"

# When performing arithmetic comparisons for testing
if (( a < b )); then
  ...
fi

# Some calculation assigned to a variable.
(( i = 10 * j + 400 ))
```

```bash
# This form is non-portable and deprecated
i=$[2 * 10]

# Despite appearances, 'let' isn't one of the declarative keywords,
# so unquoted assignments are subject to globbing wordsplitting.
# For the sake of simplicity, avoid 'let' and use (( … ))
let i="2 + 2"

# The expr utility is an external program and not a shell builtin.
i=$( expr 4 + 4 )

# Quoting can be error prone when using expr too.
i=$( expr 4 '*' 4 )
```

抛开风格方面的考虑，shell 的内置算术比 `expr` 快很多倍。

当使用变量时，在 `$(( ... ))` 中不需要 `${var}`（和 `$var`）形式。Shell 知道为您查找 `var`，省略 `${...}` 能够让代码更简洁。这与之前关于始终使用大括号的规则略有不同，因此这只是一个建议。

```bash
# N.B.: Remember to declare your variables as integers when
# possible, and to prefer local variables over globals.
local -i hundred=$(( 10 * 10 ))
declare -i five=$(( 10 / 2 ))

# Increment the variable "i" by three.
# Note that:
#  - We do not write ${i} or $i.
#  - We put a space after the (( and before the )).
(( i += 3 ))

# To decrement the variable "i" by five:
(( i -= 5 ))

# Do some complicated computations.
# Note that normal arithmetic operator precedence is observed.
hr=2
min=5
sec=30
echo $(( hr * 3600 + min * 60 + sec )) # prints 7530 as expected
```

## 命名约定

### 函数名

使用小写字母，并用下划线分隔单词。使用双冒号 `::` 分隔库。函数名称之后需要有圆括号。关键词 `function` 是可选的，但必须在一个项目中保持一致。

如果您正在编写单个函数，请使用小写字母并使用下划线分隔单词。 如果您正在编写一个包，请使用 `::` 分隔包名。大括号必须和函数名称位于同一行（与 Google 的其他语言一样），并且函数名称和括号之间不能有空格。

```bash
# Single function
my_func() {
  ...
}

# Part of a package
mypackage::my_func() {
  ...
}
```

当函数名称后面存在 "()" 时，`function` 关键字就无关紧要了，但可以促进函数的快速识别。

### 变量名

循环的变量名称应该与您正在循环的任何变量类似地命名。

```bash
for zone in "${zones[@]}"; do
  something_with "${zone}"
done
```

### 常量和环境变量名

使用大写字母，并用下划线分隔单词，在文件顶部声明。

常量和任何导出到环境中的东西都应该大写。

```bash
# Constant
readonly PATH_TO_FILES='/some/path'

# Both constant and environment
declare -xr ORACLE_SID='PROD'
```

在第一次设置时有些就变成了常量（例如，通过 getopts）。 因此，可以在 getopts 中或基于条件来设置常量，但之后应该立即将其设为只读。 为了清楚起见，建议使用 `readonly` 或 `export` 而不是等效的 `declare` 命令。

```bash
VERBOSE='false'
while getopts 'v' flag; do
  case "${flag}" in
    v) VERBOSE='true' ;;
  esac
done
readonly VERBOSE
```

### 源文件名

小写，如果需要的话，使用下划线分隔单词。

这是为了和 Google 中的其他代码风格保持一致：`maketemplate` 或 `make_template` 而不是 `make-template`。

### 只读变量

使用 `readonly` 或 `declare -r` 来确保变量只读。

由于全局变量在 shell 中被广泛使用，所以在使用它们时捕获错误是很重要的。当您声明一个只读变量时，请显式声明。

```bash
zip_version="$(dpkg --status zip | grep Version: | cut -d ' ' -f 2)"
if [[ -z "${zip_version}" ]]; then
  error_message
else
  readonly zip_version
fi
```

### 使用本地变量

使用 `local` 声明特定于函数的变量。声明和赋值应该在不同行。

通过在声明局部变量时使用 `local`，确保局部变量只在函数及其子函数内部可见。这样可以避免污染全局名称空间和不经意间设置可能在函数外部具有重要意义的变量。

当赋值的值由命令替换提供时，声明和赋值必须是单独的语句；因为内置的 `local` 不会从命令替换传递退出码。

```bash
my_func2() {
  local name="$1"

  # Separate lines for declaration and assignment:
  local my_var
  my_var="$(my_func)"
  (( $? == 0 )) || return

  ...
}
```

```bash
my_func2() {
  # DO NOT do this:
  # $? will always be zero, as it contains the exit code of 'local', not my_func
  local my_var="$(my_func)"
  (( $? == 0 )) || return

  ...
}
```

### 函数位置

将所有函数放在文件中常量的下面。不要在函数之间隐藏可执行代码。这样做会使代码难以理解，并在调试时导致令人讨厌的意外。

如果您有函数，请将它们全部放在文件顶部附近。只有头文件（includes）、`set` 语句和设置常量可以在声明函数之前完成。

### 主函数 main

对于包含至少一个其他函数的足够长的脚本，需要一个名为 `main` 的函数。

为了方便找到程序的开头，把主程序放在一个叫做 main 的函数中，作为最底层的函数。这提供了与代码库其余部分的一致性，并允许您定义更多变量为局部变量（如果主代码不是函数，则无法这样做）

这提供了与代码库其余部分的一致性，并允许您将更多变量定义为 `local`（如果主代码不是函数则不能这样做）。文件中最后一个非注释行应该是对 `main` 函数的调用：

````bash
main "$@"
````

显然，对于只是线性流的短脚本， `main` 有些矫枉过正，因此不是必需的。

## 调用命令

### 检查返回值

总是检查返回值并给出有参考价值的返回值。

对于非管道命令，使用 `$?` 或者通过 `if` 语句直接检查以保持简洁。

例如：

````bash
if ! mv "${file_list[@]}" "${dest_dir}/"; then
  echo "Unable to move ${file_list[*]} to ${dest_dir}" >&2
  exit 1
fi

# Or
mv "${file_list[@]}" "${dest_dir}/"
if (( $? != 0 )); then
  echo "Unable to move ${file_list[*]} to ${dest_dir}" >&2
  exit 1
fi
````

Bash 还有一个 `PIPESTATUS` 变量，它允许检查来自管道所有部分的返回代码。如果只需要检查整个管道是成功还是失败，则以下的方法是可以接受的:

````bash
tar -cf - ./* | ( cd "${dir}" && tar -xf - )
if (( PIPESTATUS[0] != 0 || PIPESTATUS[1] != 0 )); then
  echo "Unable to tar files to ${dir}" >&2
fi
````

但是，由于 `PIPESTATUS` 将在您执行任何其他命令后立即被覆盖，如果您需要根据错误在管道中发生的位置采取不同的操作，则需要在运行该命令后立即将 `PIPESTATUS` 赋值给另一个变量（不要忘记 `[` 是一个命令，会清除 `PIPESTATUS`）。

```bash
tar -cf - ./* | ( cd "${DIR}" && tar -xf - )
return_codes=( "${PIPESTATUS[@]}" )
if (( return_codes[0] != 0 )); then
  do_something
fi
if (( return_codes[1] != 0 )); then
  do_something_else
fi
```

### 内置命令与外部命令

如果可以在调用 shell 内置命令和调用单独的程序之间进行选择，请选择内置命令。

我们更喜欢使用内置命令，例如 `bash(1)` 中的 *Parameter Expansion* 函数，因为它更健壮、更可移植（尤其是与 `sed` 这样的命令比较）。

例如：

````bash
# Prefer this:
addition=$(( X + Y ))
substitution="${string/#foo/bar}"
````

````bash
# Instead of this:
addition="$(expr "${X}" + "${Y}")"
substitution="$(echo "${string}" | sed -e 's/^foo/bar/')"
````

## 总结

使用常识并保持一致。

编辑代码时，花点时间看看项目中的其它代码，并熟悉其风格。

风格指南的重点在于提供一个通用的编程规范，这样大家可以把精力集中在实现内容而不是表现形式上。我们展示的是一个总体的风格规范，但局部风格也很重要，如果你在一个文件中新加的代码和原有代码风格相去甚远，这就破坏了文件本身的整体美观，也会打乱读者在阅读代码时的节奏，所以要尽量避免。

好了，关于编码风格写的够多了；代码本身才更有趣。尽情享受吧！
