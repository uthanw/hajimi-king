### READMEv2 — queries.txt 读取路径说明

- **最终读取文件**: 如果 `QUERIES_FILE` 是相对路径（默认 `queries.txt`），程序读取的实际文件是 `[DATA_PATH]/[QUERIES_FILE]`。
  - 默认值：`DATA_PATH=/app/data`，`QUERIES_FILE=queries.txt` → 读取 `/app/data/queries.txt`
  - 如果 `QUERIES_FILE` 是绝对路径（以 `/` 开头），则直接使用该绝对路径（不会再拼接 `DATA_PATH`）。
- **文件不存在时**: 会自动创建并写入示例查询内容。
- **Docker Compose 情况**: 挂载 `./data:/app/data` 后，宿主机上的文件路径为 `./data/queries.txt`。

### 代码依据（关键片段）

```1:183:/workspace/common/config.py
# ... existing code ...
class Config:
    DATA_PATH = os.getenv('DATA_PATH', '/app/data')
    # ... existing code ...
    QUERIES_FILE = os.getenv("QUERIES_FILE", "queries.txt")
# ... existing code ...
```

```214:233:/workspace/utils/file_manager.py
    def load_search_queries(self, queries_file_path: str) -> List[str]:
        """从文件中加载搜索查询列表"""
        queries = []
        full_path = os.path.join(self.data_dir, queries_file_path)

        if not os.path.exists(full_path):
            self._create_default_queries_file(full_path)

        try:
            with open(full_path, "r", encoding="utf-8") as f:
                for line in f:
                    line = line.strip()
                    if line and not line.startswith('#'):
                        queries.append(line)
        except Exception as e:
            logger.error(f"Failed to read {full_path}: {e}")
            logger.info("Using empty query list")

        return queries
```

```488:489:/workspace/utils/file_manager.py
file_manager = FileManager(Config.DATA_PATH)
checkpoint = file_manager.load_checkpoint()
```

```451:466:/workspace/utils/file_manager.py
    def _create_default_queries_file(self, queries_file: str) -> None:
        """创建默认的查询文件"""
        try:
            os.makedirs(os.path.dirname(queries_file), exist_ok=True)
            with open(queries_file, "w", encoding="utf-8") as f:
                f.write("# GitHub搜索查询配置文件\n")
                f.write("# 每行一个查询语句，支持GitHub搜索语法\n")
                f.write("# 以#开头的行为注释，空行会被忽略\n")
                f.write("\n")
                f.write("# 基础API密钥搜索\n")
                f.write("AIzaSy in:file\n")
                f.write("AIzaSy in:file filename:.env\n")
                f.write("AIzaSy in:file filename:env.example\n")
            logger.info(f"Created default queries file: {queries_file}")
        except Exception as e:
            logger.error(f"Failed to create default queries file {queries_file}: {e}")
```

### 快速使用

- **默认（无需改动）**
  - 创建宿主机数据目录并放入查询：
    ```bash
    mkdir -p data
    echo "AIzaSy in:file" > data/queries.txt
    ```
  - 使用 Docker Compose（已挂载 `./data:/app/data`）启动。

- **自定义路径**
  - 通过环境变量覆盖：
    - `DATA_PATH`：数据目录（容器内路径）
    - `QUERIES_FILE`：查询文件名或绝对路径
  - 示例（Compose 环境变量）：
    ```yaml
    environment:
      - DATA_PATH=/app/data
      - QUERIES_FILE=queries.txt   # 或 /app/data/custom/queries.prod.txt
    volumes:
      - ./data:/app/data
    ```

### 结论
- 代码明确：查询文件的最终路径为 `os.path.join(DATA_PATH, QUERIES_FILE)`，默认即 `/app/data/queries.txt`；若 `QUERIES_FILE` 为绝对路径则直接使用该路径。文件缺失会自动创建并填充示例内容。