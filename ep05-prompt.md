# 任务：从零实现 Rust mini-redis 服务器

从零实现一个简易 Redis 服务器，支持 TCP 并发连接、键值存储、TTL 过期、AOF 持久化。

## 功能要求

### 1. TCP 服务器
- 监听 `127.0.0.1:6379`
- 支持多客户端并发连接（使用 tokio async runtime）
- 每个连接独立处理，互不阻塞

### 2. 命令协议（简化文本协议）

客户端发送一行文本命令，服务器返回一行响应。命令不区分大小写。

**值支持双引号包裹**：如果 value 包含空格，客户端会用双引号包裹，例如 `SET key "hello world"`。服务器需要正确解析引号内的内容作为完整值（去掉引号本身）。不含空格的值可以不带引号。

| 命令 | 格式 | 响应 |
|------|------|------|
| PING | `PING` | `+PONG` |
| SET | `SET key value` | `+OK` |
| SET with TTL | `SET key value EX seconds` | `+OK` |
| GET | `GET key` | `+value` 或 `$-1`（不存在/已过期） |
| DEL | `DEL key [key ...]` | `:{N}`（成功删除的数量） |
| INCR | `INCR key` | `:{new_value}`（原子自增 1，key 不存在则从 0 开始） |
| EXPIRE | `EXPIRE key seconds` | `:1`（成功）或 `:0`（key 不存在） |
| KEYS | `KEYS pattern` | 每行一个 key，最后一行 `+END`。pattern 支持 `*` 通配 |
| 无效命令 | 任意无法解析的输入 | `-ERR unknown command` |

**INCR 说明**：
- 如果 key 不存在，视为 0，执行后返回 `:1`
- 如果 key 的值不是合法整数，返回 `-ERR value is not an integer`
- INCR 必须是原子操作，并发 INCR 不能丢失计数

### 3. TTL 过期
- `SET key value EX 10` 表示 key 在 10 秒后过期
- 惰性删除：GET 时检查是否过期，过期则返回 `$-1` 并删除
- 主动清理：后台任务每 1 秒扫描一次，删除已过期的 key

### 4. AOF 持久化
- 写命令（SET / DEL / EXPIRE / INCR）执行成功后，追加一行到 `appendonly.aof` 文件
- **格式**：`<unix_timestamp_ms> <command>`，例如 `1714900000000 SET foo bar EX 60`
  - 时间戳为写入时的 Unix 毫秒时间戳（`SystemTime::now()`）
  - 用于重放时精确还原 TTL 过期时间
- **重放规则**：
  - 启动时如果 `appendonly.aof` 存在，逐行读取并重放
  - 对于带 EX 的 SET 命令：根据记录的时间戳计算剩余 TTL，如果已过期则跳过
  - 例如：记录 `1714900000000 SET foo bar EX 60`，当前时间 `1714900050000`（50 秒后），剩余 TTL = 10 秒
  - 如果剩余 TTL ≤ 0，该条记录直接跳过不恢复
- 优雅关闭：收到 Ctrl+C（SIGINT）时，确保 AOF 缓冲区刷盘后再退出

### 5. 数据存储
- 使用 `Arc<RwLock<HashMap<String, Entry>>>` 作为共享状态
- `Entry` 结构包含 `value: String` 和 `expires_at: Option<Instant>`
- INCR 操作需要对单个 key 做原子读写，注意锁粒度

## 约束

- 必须使用 `tokio` 运行时（`#[tokio::main]`）
- **不允许使用 `unsafe`**
- 代码总行数不超过 **800 行**（含注释）
- 每个公开函数/结构体必须有 **doc comment**（`///`）
- 项目结构：单 crate，`src/main.rs` 为入口，可拆分模块
- 仅允许使用以下依赖：`tokio`（full features）、`signal-hook`（可选）

## 项目初始化

项目已创建好 `Cargo.toml`，你只需要编写 `src/` 下的代码。

## 验证

完成后依次运行：
```bash
cargo build --release
cargo clippy -- -D warnings
```

两个命令都通过即为代码提交。功能验证由外部测试脚本完成。
