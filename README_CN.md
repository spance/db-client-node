# DB-Client-Node

<p>
<a href="https://github.com/spance/db-client-node"><img alt="GitHub Release" src="https://img.shields.io/github/v/release/spance/db-client-node?style=social"/></a>
<a href="https://marketplace.dify.ai/plugins/spance/db_client_node"><img alt="Dify Marketplace" src="https://img.shields.io/badge/Dify%20Marketplace-DB_Client_Node-blue"/></a>
</p>

**简体中文** | [English](./README.md) 

[DB-Client-Node](https://github.com/spance/db-client-node) 是为 Dify 工作流设计的插件工具，提供数据库操作节点。目前支持 PostgreSQL 数据库。

![DB-Client-Node demo](https://f001.backblazeb2.com/file/static/dbcn-demo.png)

## 特性
- 通过 Dify 的 `credentials_for_provider` 机制安全管理数据库连接信息。
- 支持参数化执行 `SELECT` 和 `INSERT` 等 DML 语句。
- 提供动态 SQL 生成及条件逻辑支持。
- 实现性能优化的批量数据插入功能。


## 安装
要在 Dify 工作流中使用 `DB-Client-Node`：
1. 通过 Dify [插件市场](https://marketplace.dify.ai/plugins/spance/db_client_node) 安装插件，或从 [发布页面](https://github.com/spance/db-client-node/releases) 手动下载版本。
2. 在 Dify 的 `插件 -> 授权` 设置中配置 PostgreSQL 凭据。


## 节点概览

### 普通客户端节点
普通客户端节点用于执行 `SELECT` 和 `INSERT` 等 DML 语句。由于 Dify 工具插件的表单界面不支持动态参数，提供了两个变体：
- **`PostgreSQL Client(5)`**: 支持最多 5 个 SQL 参数。
- **`PostgreSQL Client(10)`**: 支持最多 10 个 SQL 参数。

两个节点除了参数数量不同外，其余功能一致。

#### 使用示例

1. **无参数执行 SQL**
   | 字段       | 填写                              |
   |------------|-----------------------------------|
   | SQL        | `SELECT * FROM messages WHERE id=123` |
   | 参数 0-N   | *留空*                           |

2. **带参数执行 SQL**
   | 字段       | 填写                              |
   |------------|-----------------------------------|
   | SQL        | `SELECT * FROM messages ` <br/> `WHERE name=$arg0 AND user=$arg1` |
   | 参数 0     | 对应 `$arg0`：输入字面值或使用 `/` 选择节点变量 |
   | 参数 1     | 对应 `$arg1`：输入字面值或使用 `/` 选择节点变量 |

3. **动态 SQL 执行**
   | 字段       | 填写                              |
   |------------|-----------------------------------|
   | SQL        | `SELECT * FROM messages ` <br/> `WHERE name=$arg0` <br/> `{% if arg1 %} AND user=$arg1 {% endif %}` |
   | 参数 0     | 对应 `$arg0`：输入字面值或使用 `/` 选择节点变量 |
   | 参数 1     | 对应 `$arg1`：输入字面值或使用 `/` 选择节点变量 |

   - 其中 `{% if %} ... {% endif %}` 语法使用 Jinja2 模板实现动态 SQL。如果条件（如 `arg1`）不成立，该部分将被排除在执行的 SQL 之外；反之则包含。

### 批量客户端节点
批量客户端节点专为批量数据插入设计，利用数据库的批提交机制减少 I/O 开销和延迟。

#### 使用示例
| 字段             | 填写                              |
|------------------|-----------------------------------|
| SQL              | `INSERT INTO messages (name, user, content) ` <br/> `VALUES (?, ?, ?)` |
| 参数对象列表     | 选择上游节点变量，传入 JSON 序列化的参数对象列表，例如 `List[List[Any]]` 或 `List[Dict[str, Any]]`。示例：<br/> - `"[[1,2,3],[5,6,7]]"` <br/> - `"[{name:1,user:2,content:3},{name:5,user:6,content:7}]"` |
| 取值的键名       | 如果参数列表为数组，则留空；如果为字典类型，提供逗号分隔的键名列表以提取值（注意顺序），例如：`name,user,content` |


## 节点输出

### 查询执行
对于 `SELECT` 语句，节点返回：
- **`data`**: 查询结果数据，按行排列。
- **`columns`**: 结果集的列名。

**示例输出：**
```json
{
    "data": [
        ["row0_column0", "column1", "column2"],
        ["row1_column0", "column1", "column2"],
        ["row2_column0", "column1", "column2"]
    ],
    "columns": ["name", "user", "content"]
}
```

### 更新执行
对于 `INSERT`、`UPDATE` 或 `DELETE` 语句，节点返回：
- **`affected_rows`**: 受影响的行数。

**示例输出：**
```json
{
    "affected_rows": 2
}
```

## 限制
- 目前仅支持 PostgreSQL 数据库。
- 普通客户端节点参数数量固定（5 或 10），受限于 Dify 表单界面。
- 动态 SQL 依赖 Jinja2 语法，可能需要用户熟悉模板语言。

## 贡献指南
欢迎贡献代码！参与步骤如下：
1. Fork 本仓库。
2. 创建功能分支（`git checkout -b feature/your-feature`）。
3. 提交更改（`git commit -m "添加您的功能"`）。
4. 推送分支（`git push origin feature/your-feature`）。
5. 提交 Pull Request。

请确保代码遵循项目风格指南并包含适当的测试。

## 许可证
本项目采用 Apache License 2.0 发布。详情请参阅 `LICENSE` 文件。

## 联系方式
如有问题或需要支持，请在仓库中提交 issue。
