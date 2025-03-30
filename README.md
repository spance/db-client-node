# DB-Client-Node

**Author:** spance  
**Version:** 0.1.45  
**Type:** Tool  

[简体中文](./README_CN.md) | **English**

`DB-Client-Node` is a plugin tool designed for Dify workflows, providing database operation nodes. It currently supports `PostgreSQL` databases.

## Features
- Securely manages database connection information through Dify’s `credentials_for_provider` mechanism.
- Supports parameterized execution of DML statements such as `SELECT` and `INSERT`.
- Provides dynamic SQL generation with conditional logic support.
- Implements performance-optimized batch data insertion.

## Installation
To use `DB-Client-Node` in a Dify workflow:
1. Install the plugin via the Dify plugin marketplace or manually download version `0.1.44` from the [release page](#).
2. Configure PostgreSQL credentials in Dify’s `Plugins -> Authorization` settings.

## Nodes Overview

### Standard Client Nodes
Standard client nodes are used to execute DML statements such as `SELECT` and `INSERT`. Due to limitations in Dify’s tool plugin form interface, which does not support dynamic parameters, two variants are provided:
- **`PostgreSQL Client(5)`**: Supports up to 5 SQL parameters.
- **`PostgreSQL Client(10)`**: Supports up to 10 SQL parameters.

These nodes are identical except for the number of supported parameters.

#### Usage Examples

1. **Executing SQL Without Parameters**
   | Field       | Value                              |
   |-------------|------------------------------------|
   | SQL         | `SELECT * FROM messages WHERE id=123` |
   | Parameter 0-N | *Leave blank*                    |

2. **Executing SQL With Parameters**
   | Field       | Value                              |
   |-------------|------------------------------------|
   | SQL         | `SELECT * FROM messages ` <br/> `WHERE name=$arg0 AND user=$arg1` |
   | Parameter 0 | Corresponding to `$arg0`: Enter a literal value or use `/` to select a node variable |
   | Parameter 1 | Corresponding to `$arg1`: Enter a literal value or use `/` to select a node variable |

3. **Executing Dynamic SQL**
   | Field       | Value                              |
   |-------------|------------------------------------|
   | SQL         | `SELECT * FROM messages ` <br/> `WHERE name=$arg0` <br/> `{% if arg1 %} AND user=$arg1 {% endif %}` |
   | Parameter 0 | Corresponding to `$arg0`: Enter a literal value or use `/` to select a node variable |
   | Parameter 1 | Corresponding to `$arg1`: Enter a literal value or use `/` to select a node variable |

   - The `{% if %} ... {% endif %}` syntax uses Jinja2 templating to enable dynamic SQL. If the condition (e.g., `arg1`) is not met, that portion is excluded from the executed SQL; otherwise, it is included.

### Batch Client Node
The batch client node is designed for bulk data insertion, utilizing the database’s batch submission mechanism to reduce I/O overhead and latency.

#### Usage Example
| Field             | Value                              |
|-------------------|------------------------------------|
| SQL               | `INSERT INTO messages (name, user, content) ` <br/> `VALUES (?, ?, ?)` |
| Parameter Object List | Select an upstream node variable containing a JSON-serialized list of parameter objects, e.g., `List[List[Any]]` or `List[Dict[str, Any]]`. Examples: <br/> - `"[[1,2,3],[5,6,7]]"` <br/> - `"[{name:1,user:2,content:3},{name:5,user:6,content:7}]"` |
| Key Names for Extraction | Leave blank if the parameter list is an array. For dictionary-type lists, provide a comma-separated list of keys to extract values (in order), e.g., `name,user,content` |

## Node Output

### Query Execution
For `SELECT` statements, the node returns:
- **`data`**: The query result data, organized by rows.
- **`columns`**: The column names of the result set.

**Example Output:**
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

### Update Execution
For `INSERT`, `UPDATE`, or `DELETE` statements, the node returns:
- **`affected_rows`**: The number of rows affected.

**Example Output:**
```json
{
    "affected_rows": 2
}
```

## Limitations
- Currently supports only PostgreSQL databases.
- The number of parameters in standard client nodes is fixed (5 or 10), limited by Dify’s form interface.
- Dynamic SQL relies on Jinja2 syntax, which may require familiarity with templating.

## Contribution Guidelines
Contributions are welcome! To participate:
1. Fork this repository.
2. Create a feature branch (`git checkout -b feature/your-feature`).
3. Commit your changes (`git commit -m "Add your feature"`).
4. Push to the branch (`git push origin feature/your-feature`).
5. Submit a Pull Request.

Please ensure your code adheres to the project’s style guidelines and includes appropriate tests.

## License
This project is released under the Apache License 2.0. See the `LICENSE` file for details.

## Contact
For questions or support, please submit an issue in the repository.
