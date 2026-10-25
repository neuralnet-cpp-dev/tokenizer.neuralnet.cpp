# tokenizer.neuralnet.cpp

neuralnet.cpp 生态的 **BPE / CharBPE 分词器**库（header-only）。

| 头文件 | 内容 |
|---|---|
| `domain_tokenizer.hpp` | 聚合入口 |
| `domain_tokenizer_base.hpp` | `Tokenizer` 基类 + 并行预分词 + 词表持久化（`load`/`save`） |
| `domain_tokenizer_bpe.hpp` | `BPETokenizer` |
| `domain_tokenizer_charbpe.hpp` | `CharBPETokenizer` + `make_tokenizer` 工厂 |

## 用法

```cpp
#include <neuralnet.cpp/domain_tokenizer.hpp>

auto tok = nn::make_tokenizer(...);   // 工厂
tok->train(...);
tok->save("vocab.nnvocab");
```

## 依赖

均以 submodule 位于 `extern/`：

- [`base.neuralnet.cpp`](https://github.com/neuralnet-cpp-dev/base.neuralnet.cpp) — `Result<T>` / 断言 / 精度 / 常量
- [`threadpool.neuralnet.cpp`](https://github.com/neuralnet-cpp-dev/threadpool.neuralnet.cpp) — 并行预分词
- [`kvrecord.neuralnet.cpp`](https://github.com/neuralnet-cpp-dev/kvrecord.neuralnet.cpp) — 词表二进制容器
- [`dataset.neuralnet.cpp`](https://github.com/neuralnet-cpp-dev/dataset.neuralnet.cpp) — 词表文件格式（`parse_nnvocab`）

```bash
git clone --recurse-submodules .../tokenizer.neuralnet.cpp
```

CMake：

```cmake
add_subdirectory(tokenizer)   # 自动拉起 extern/{base,threadpool,kvrecord,dataset}
target_link_libraries(app PRIVATE neuralnet::tokenizer)
```
