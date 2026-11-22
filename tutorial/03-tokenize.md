# Day 3：分词

**目标**：把一行文本变成一串 Token，并打印出来。

用时：约 2 小时。

---

## 3.1 为什么需要 Token

直接切分只能得到字符串数组，但后面需要知道：

- 片段在原句的哪个位置（用来还原原文）
- 它是不是标点（用来判断切块边界）

所以用一个结构体把这两件事捆起来：

```cpp
struct Token {
    std::string text;    // 片段内容
    size_t pos;          // 在原串里的起始位置
    bool isPunct;        // 是不是标点
};
```

> `size_t` 是专门表示大小/下标的无符号整数，和 `string::size()`、`vector::size()` 的类型一致。
> 用 `int` 会触发 C4267 警告。

---

## 3.2 三个新东西

| 写法 | 作用 |
|---|---|
| `struct Token { ... };` | 把几个变量捆成一个新类型 |
| `std::vector<Token> v;` | 可变长数组 |
| `v.push_back(x);` | 往末尾加一个元素 |

两个习惯：

```cpp
vector<Token> tokenize(const string& s);
```

- `&` 传引用：不复制整个字符串
- `const`：函数内部不改它

---

## 3.3 切分规则（v1）

| 字符 | 处理 |
|---|---|
| 空格 / 制表符 | 结束当前词 |
| 字母、数字 | 累积到当前词 |
| `'` 或 `-` | **前后都是字母**才并入词内（`don't`、`well-known`），否则算标点 |
| 其它可见符号（`,.!?;:`） | 结束当前词，自己单独成一个 token |
| 非 ASCII（中文、emoji） | 暂时跳过 |

---

## 3.4 参考实现

```cpp
#include <cctype>
#include <string>
#include <vector>

using namespace std;

struct Token {
    string text;
    size_t pos;
    bool isPunct;
};

vector<Token> tokenize(const string& s) {
    vector<Token> out;
    string buf;          // 正在累积的词
    size_t start = 0;    // 这个词从哪儿开始

    for (size_t i = 0; i < s.size(); ) {
        unsigned char c = static_cast<unsigned char>(s[i]);

        if (c >= 0x80) { ++i; continue; }              // TODO: 中文以后处理

        if (isspace(c)) {                              // ① 空白：结束当前词
            if (!buf.empty()) { out.push_back({buf, start, false}); buf.clear(); }
            ++i;
            continue;
        }

        if (isalnum(c)) {                              // ② 字母数字：累积
            if (buf.empty()) start = i;
            buf += s[i];
            ++i;
            continue;
        }

        if (c == '\'' || c == '-') {                   // ③ 撇号/连字符
            bool nextIsLetter = (i + 1 < s.size()) &&
                                isalpha(static_cast<unsigned char>(s[i + 1]));
            if (!buf.empty() && nextIsLetter) {
                buf += s[i];
                ++i;
                continue;
            }
        }

        // ④ 标点：先收尾当前词，再自己成 token
        if (!buf.empty()) { out.push_back({buf, start, false}); buf.clear(); }
        out.push_back({string(1, s[i]), i, true});
        ++i;
    }

    if (!buf.empty()) out.push_back({buf, start, false});   // 收尾
    return out;
}
```

**最后一行是循环收尾**——Day 2 那个「最后一个词」的坑，这里又出现了一次。

---

## 3.5 在 main 里使用

```cpp
while (getline(in, line)) {
    if (!line.empty() && line.back() == '\r') line.pop_back();

    vector<Token> tokens = tokenize(line);
    for (size_t k = 0; k < tokens.size(); ++k) {
        cout << k << " [" << tokens[k].text << "]"
             << (tokens[k].isPunct ? " 标点" : "") << endl;
    }
    cout << "---- 本行 " << tokens.size() << " 个 token ----" << endl;
}
```

---

## 3.6 必须通过的测试

| 输入 | 期望 token 数 | 内容 |
|---|---|---|
| `Hello, world!` | 4 | `Hello` `,` `world` `!` |
| `don't stop` | 2 | `don't` `stop` |
| `well-known fact` | 2 | `well-known` `fact` |
| `a  b`（两个空格） | 2 | `a` `b` |
| `   `（全空格） | 0 | — |
| `End.` | 2 | `End` `.` |

---

## 3.7 常见错误

| 现象 | 原因 |
|---|---|
| 最后一个词丢了 | 忘了循环后的收尾 |
| `don` 和 `t` 分开 | 撇号分支没写，或判断顺序错 |
| 出现空 token | 空白处没检查 `buf.empty()` |
| 编译报 `Token` 未定义 | `struct` 定义写在了函数后面 |
| `isalnum` 报错 | 忘了 `#include <cctype>` |

---

## 3.8 验收

`"Hello, world!"` 应输出：

```
0 [Hello]
1 [,] 标点
2 [world]
3 [!] 标点
---- 本行 4 个 token ----
```

---

## 3.9 关于中英文混排

现在非 ASCII 字节被直接跳过（`if (c >= 0x80) { ++i; continue; }`）。

以后要支持中文时，把这一行换成一个 `charLen()` 函数：

```cpp
size_t charLen(const string& s, size_t i) {
    unsigned char c = static_cast<unsigned char>(s[i]);
    if (c < 0x80)         return 1;
    if ((c >> 5) == 0x6)  return 2;
    if ((c >> 4) == 0xE)  return 3;   // 中文
    if ((c >> 3) == 0x1E) return 4;   // emoji
    return 1;
}
```

循环改成 `i += charLen(s, i)` 即可。**这是预留的扩展点**，现在不用做。
