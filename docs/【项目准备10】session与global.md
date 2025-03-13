# 数据库中的session和global分别是什么

在数据库中，`SESSION` 和 `GLOBAL` 是两个与配置和作用域相关的概念，主要用于管理数据库连接、变量或权限的生效范围。它们的具体含义因数据库系统（如 MySQL、PostgreSQL、SQL Server 等）略有不同，但核心思想一致。

---

### **1. SESSION（会话）**
**定义**：  
`SESSION` 是指用户与数据库服务器之间的一个独立会话（连接）。每个会话代表一次客户端与数据库的交互过程，包含用户的身份信息、当前执行的操作状态、临时变量等。会话的配置（如字符集、时区、事务隔离级别等）**仅对当前会话有效**。

**典型应用场景**：  
- **用户权限**：会话中的用户身份决定了其能访问哪些数据（如通过 `GRANT` 授予的权限）。  
- **本地变量**：在会话中设置的变量（如 MySQL 的 `@@session.variable`）仅对当前会话生效。  
- **事务控制**：会话维护事务的开始、提交或回滚状态。  

**示例（MySQL）**：  
```sql
-- 查看当前会话的字符集
SHOW VARIABLES LIKE 'character_set%';

-- 在会话中临时修改字符集（仅对当前会话生效）
SET NAMES utf8mb4;

-- 查看全局字符集（所有会话的默认值）
SHOW GLOBAL VARIABLES LIKE 'character_set%';
```

---

### **2. GLOBAL（全局）**
**定义**：  
`GLOBAL` 是指数据库服务器实例级别的配置，作用于所有连接到该服务器的会话。全局配置通常是数据库管理员设置的持久化参数（如默认字符集、最大连接数等），**所有新会话默认继承这些配置**，但会话可以通过 `SESSION` 覆盖它们。

**典型应用场景**：  
- **服务器参数**：如最大并发连接数 (`max_connections`)、默认存储引擎 (`storage_engine`)。  
- **全局变量**：在 MySQL 中通过 `SET GLOBAL` 修改的变量会影响所有新会话（需管理员权限）。  
- **安全策略**：如全局权限表 (`mysql.user`) 控制所有用户的登录和操作权限。  

**示例（MySQL）**：  
```sql
-- 修改全局变量（需要 SUPER 权限）
SET GLOBAL max_connections = 300;

-- 查看全局变量
SHOW GLOBAL VARIABLES LIKE 'max_connections';
```

---

### **3. SESSION vs GLOBAL 的核心区别**
| **特性**       | **SESSION**                     | **GLOBAL**                     |
|------------------|----------------------------------|----------------------------------|
| **生效范围**     | 仅当前会话                       | 整个数据库实例的所有会话         |
| **优先级**       | 覆盖 GLOBAL 的默认配置            | 默认值，可被子会话覆盖           |
| **生命周期**     | 随会话结束（断开连接）失效       | 持久化存储，重启后仍然有效       |
| **配置权限**     | 普通用户可修改部分会话变量       | 通常需要管理员权限               |

---

### **4. 实际应用示例**
#### **场景 1：修改字符集**  
- **全局设置**（管理员）：  
  ```sql
  SET GLOBAL default_charset = utf8mb4;
  ```  
  所有新会话将默认使用 `utf8mb4`，但已有会话不受影响。  

- **会话设置**（普通用户）：  
  ```sql
  SET NAMES utf8mb4; -- 等同于 SET SESSION character_set_client=utf8mb4
  ```  
  仅当前会话使用 `utf8mb4`，断开后失效。

#### **场景 2：事务隔离级别**  
- **全局默认**（如 PostgreSQL）：  
  ```sql
  ALTER SYSTEM SET transaction_isolation_level = 'read committed';
  ```  
  所有新会话默认使用 `READ COMMITTED` 隔离级别。  

- **会话覆盖**：  
  ```sql
  BEGIN ISOLATION LEVEL SERIALIZABLE;
  -- 当前会话使用 SERIALIZABLE 隔离级别
  COMMIT;
  ```

---

### **5. 注意事项**
1. **权限控制**：  
   - 修改 `GLOBAL` 变量通常需要管理员权限（如 MySQL 的 `SUPER` 权限）。  
   - 普通用户只能修改自己的 `SESSION` 变量。  

2. **配置持久化**：  
   - `GLOBAL` 变量的修改通常是持久的（重启后生效），具体取决于数据库实现。  
   - 部分数据库（如 PostgreSQL）将全局配置存储在配置文件（`postgresql.conf`）中。  

3. **覆盖规则**：  
   - `SESSION` 变量会覆盖 `GLOBAL` 的默认值，但某些变量可能无法覆盖（如用户权限）。  

---

### **总结**
- **SESSION**：关注单个会话的临时状态和个性化配置。  
- **GLOBAL**：关注数据库实例的全局默认行为和持久化设置。  

理解两者的区别有助于更好地管理数据库性能、安全和用户体验。在实际开发中，应根据需求权衡使用会话级配置和全局配置。
