# tokenizer.neuralnet.cpp

neuralnet.cpp 生态的 **BPE / CharBPE 分词器**库（header-only）。

| 头文件 | 内容 |
|---|---|
| `domain_tokenizer.hpp` | 聚合入口（include 它即可） |
| `domain_tokenizer_base.hpp` | `Tokenizer` 基类 + 并行预分词（`pre_tokenize`）+ 词表持久化（`to_kvrec`/`load_from_kvrec`/`save`/`load`）+ 多轮对话 prompt 构建（`chat_prompt`）+ `encode_docs_parallel` |
| `domain_tokenizer_bpe.hpp` | `BPETokenizer`（GPT 风格字节对编码） |
| `domain_tokenizer_charbpe.hpp` | `CharBPETokenizer` + `make_tokenizer` 工厂 + `load_tokenizer_from_{string,bytes,file}` |

## API 速览

```cpp
#include <neuralnet.cpp/domain_tokenizer.hpp>

// ── 训练 ──────────────────────────────────────────────────────
nn::BPETokenizer tok;
nn::BPETokenizer::Config cfg;
cfg.vocab_size = 320;
cfg.min_freq   = 1;
cfg.threads    = 0;                       // 0 = 自动（线程池全部核心）
cfg.log        = [](std::string_view) {}; // 训练日志回调
auto tr = tok.train(corpus, cfg);         // Result<void>

// ── 编解码 ────────────────────────────────────────────────────
std::vector<std::size_t> ids = tok.encode("hello world");
std::string text = tok.decode(ids);       // encode∘decode 字节级往返一致
std::size_t n = tok.vocab_size();

// 批量文档并行编码（保序：任意并行度与顺序 encode 逐字节一致）
auto per_doc = nn::encode_docs_parallel(tok, docs);

// ── 对话标记 ──────────────────────────────────────────────────
tok.has_dialogue_markers();
tok.assistant_marker_id(); tok.end_assistant_marker_id();   // 及 system/user 等
std::vector<std::size_t> prompt = tok.chat_prompt(
    {{/*user=*/true, "hi", /*tokens=*/{}}, {false, "hello", {}}},
    /*system_prompt=*/"", /*max_tokens=*/0);

// ── 词表持久化（.nnvocab，kvrec v2）────────────────────────────
tok.save("vocab.nnvocab");                          // Result<void>
nn::KeyValueRecord rec = tok.to_kvrec();            // 模型内嵌词表字节来源
auto loaded = nn::load_tokenizer_from_file("vocab.nnvocab");   // Result<unique_ptr<Tokenizer>>
auto made   = nn::make_tokenizer("bpe");            // 按类型字符串造工厂实例

// ── CharBPE（字符级 BPE）───────────────────────────────────────
nn::CharBPETokenizer cbpe;
cbpe.train(corpus, cbpe.Config{/*...*/});
```

## 用法 / CMake

```cmake
add_subdirectory(tokenizer)   # 自动拉起 extern/{base,threadpool,kvrecord,dataset}
target_link_libraries(app PRIVATE neuralnet::tokenizer)   # 提供 include + C++23
```

## 依赖

均以 submodule 位于 `extern/`：

- [`base.neuralnet.cpp`](https://github.com/neuralnet-cpp-dev/base.neuralnet.cpp) — `Result<T>` / 断言 / 精度 / 常量
- [`threadpool.neuralnet.cpp`](https://github.com/neuralnet-cpp-dev/threadpool.neuralnet.cpp) — 并行预分词（`parallel_for_samples`）
- [`kvrecord.neuralnet.cpp`](https://github.com/neuralnet-cpp-dev/kvrecord.neuralnet.cpp) — 词表二进制容器
- [`dataset.neuralnet.cpp`](https://github.com/neuralnet-cpp-dev/dataset.neuralnet.cpp) — 词表文件格式（`parse_nnvocab`/`write_nnvocab`）+ `.nndataset` 集成

```bash
git clone --recurse-submodules .../tokenizer.neuralnet.cpp
# 或已 clone 后：git submodule update --init --recursive
```

## 构建与测试

```bash
git submodule update --init --recursive
cmake -B build
cmake --build build
ctest --test-dir build --output-on-failure
```

| 测试 | 覆盖 |
|---|---|
| `tokenizer_tokenizer_test` | `pre_tokenize` 手写状态机 ↔ `std::regex` 参考逐字节对拍（缩写/数字/空白/标点/非 ASCII/标记边界 + 随机字节）；标记边界训练后仍按单 token 识别；encode∘decode 字节级往返；并行 encode 确定性（任意并行度逐字节一致） |
| `tokenizer_nnvocab_test` | 旧 JSON 词表 → `.nnvocab` → 加载语义一致（vocab/merges/markers/编解码）；训练态 save/load 往返逐位一致（BPE + CharBPE）；旧 `.json` 词表明确拒绝；词表镜像字节直载（v5 JSON / v6 kvrec） |
| `tokenizer_dataset_integration_test` | 真实 BPE 训练 → `encode_docs_parallel` 保序 → `mark_assistant_span` 掩码与全局扫描逐位一致 → 写读 `.nndataset` 逐位 round-trip（all/assistant 双模式、空 doc、train/test 双子集）→ `.nnvocab` 独立加载 |

被父工程 `add_subdirectory` 消费时测试**默认不构建**（`NN_BUILD_TESTS` 默认值 =
是否顶层工程）；显式 `-DNN_BUILD_TESTS=ON` 可在嵌入时强制打开。

许可证：LGPL-3.0（见 [LICENSE](LICENSE)）。
