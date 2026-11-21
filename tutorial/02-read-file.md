# Day 2：读文件

**目标**：写一个程序，读入一个 txt 文件，输出「行数 / 字符数 / 单词数」，并打印带行号的预览。

用时：约 2 小时。

---

## 2.0 准备文件（5 分钟）

1. 在 `D:\cpp\ChunkReader\` 下新建文件夹 `data`
2. 新建 `sample.txt`，粘贴下面四行：

```
Reading is often described as a way to escape, yet the best books do something more useful.
A sentence can hold a memory we did not know we had.
The world is loud.
Reading is a way of listening beneath it.
```

> 文件会出现在哪，取决于你怎么建的。用 VS 的「文件 → 新建 → 文件」保存到 `D:\cpp\ChunkReader\data\sample.txt` 最稳。

---

## 2.1 今天要学的五个东西

| 写法 | 作用 |
|---|---|
| `#include <fstream>` | 引入文件流库 |
| `ifstream in("路径");` | 打开文件用于**读**（`ofstream` 是写） |
| `if (!in) { ... }` | 检查是否打开成功 |
| `while (getline(in, line))` | 读一整行；读到文件末尾返回 false，循环自动结束 |
| `line.size()` | 这一行的字符数 |

**为什么用 `getline` 而不是 `in >> line`**：`>>` 遇到空格就停，一行会被拆成很多个词；`getline` 才读整行。

---

## 2.2 Step 1：打开并检查（照抄）

```cpp
#include <iostream>
#include <fstream>
#include <string>
using namespace std;

int main() {
    ifstream in("data/sample.txt");
    if (!in) {
        cout << "打开文件失败，检查路径: data/sample.txt" << endl;
        return 1;
    }

    cout << "打开成功" << endl;
    return 0;
}
```

`Ctrl+Shift+B` 生成 → `Ctrl+F5` 运行。看到「打开成功」再往下。

> 如果这里就失败了，百分之百是路径问题：文件不在 `D:\cpp\ChunkReader\data\` 下。

---

## 2.3 Step 2：逐行读，统计行数和字符数（你写）

在 `cout << "打开成功"` 之后加循环：

```cpp
string line;
int lines = 0;
int chars = 0;

while (getline(in, line)) {
    ++lines;
    chars += line.size();     // size() 返回的是字节数
}

cout << "行数: " << lines << endl;
cout << "字符数: " << chars << endl;
```

**要点**

- `getline` 把换行符吃掉，不放进 `line`，所以 `chars` 里不含换行
- 循环结束时 `lines` 就是总行数
- 文件最后一行如果没有换行，也会被读到

---

## 2.4 Step 3：打印带行号的预览（你写）

在同一个循环里加：

```cpp
int shown = 0;                 // 已经打印了几行
...
while (getline(in, line)) {
    ++lines;
    chars += line.size();

    if (shown < 3) {           // 只打印前 3 行
        cout << "第 " << (lines) << " 行 (" << line.size() << " 字符): ";
        cout << line << endl;
        ++shown;
    }
}
```

---

## 2.5 挑战：统计单词数（可选）

思路：单词数 = 空格数 + 1，但要处理空行和连续空格。

最简单可行的写法（够用就行）：

```cpp
int words = 0;
bool inWord = false;
for (char c : line) {
    if (c == ' ' || c == '\t') {
        inWord = false;
    } else if (!inWord) {
        inWord = true;
        ++words;
    }
}
```

放在循环内部，每行处理一次。

---

## 2.6 验收标准

对上面那四行文本，正确输出应该是：

```
第 1 行 (91 字符): Reading is often described as a way to escape, yet the best books do something more useful.
第 2 行 (52 字符): A sentence can hold a memory we did not know we had.
第 3 行 (18 字符): The world is loud.
第 4 行 (41 字符): Reading is a way of listening beneath it.
行数: 4
字符数: 202
单词数: 41
```

### 如果你的数字对不上

| 你得到 | 原因 |
|---|---|
| 字符数 **206** | 文件是 CRLF 换行，每行多一个 `\r`，共多 4 个 |
| 字符数 **206 且行数 4** | 同上，属正常现象，见 2.7 |
| 行数 1 | 用了 `in >> line`，或者整个文件没有换行 |
| 打开失败 | 路径不对，文件不在 `D:\cpp\ChunkReader\data\` |

---

## 2.7 必须理解的坑：`\r`

Windows 的换行是两个字符：`\r\n`（回车 + 换行）。

`getline` 只吃掉 `\n`，把 `\r` 留在 `line` 末尾。所以：

- 用 VS 或记事本保存的 txt，**大概率是 CRLF**，字符数会多 4
- 第 4 行打印出来后面可能粘着一个看不见的字符

**这不影响今天验收**（202 或 206 都算对），但你必须在报告里知道这件事——后面分词时，`"it.\r"` 会让你的标点判断出错。

修法（今天不用改，知道有这回事就行）：

```cpp
if (!line.empty() && line.back() == '\r') line.pop_back();
```

---

## 2.8 常见问题

| 现象 | 原因 |
|---|---|
| 「打开文件失败」 | 相对路径是相对**工作目录**（`D:\cpp\ChunkReader\`），不是 `main.cpp` 的位置 |
| 编译报 `ifstream 未定义` | 忘了 `#include <fstream>` |
| 编译报 `string 未定义` | 忘了 `#include <string>` |
| 循环只跑一次 | 用了 `in >> line`，应改 `getline(in, line)` |
| 字符数特别大 | 文件其实是 UTF-8 中文，`size()` 数的是字节（一个汉字 3 字节） |
| 想读第二遍读不到 | 文件指针到末尾了，需要 `in.clear(); in.seekg(0);` |

---

## 2.9 今天的产出

改完的 `main.cpp` 应该有 40 行左右。这就是将来 `io.cpp` 的雏形。

**Day 3 会用到它**：把这段读文件的代码拆成独立函数，放进 `io.cpp`。
