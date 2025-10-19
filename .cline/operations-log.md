## 操作日志 - 修复 PostgreSQL DOUBLE 类型错误

时间：2025-10-19 22:09:08

### 问题描述

- APScheduler 表创建失败，错误：`type "double" does not exist`
- 问题位置：`ruoyi-fastapi-backend/module_admin/entity/do/job_do.py` 中的 `ApschedulerJobs` 类
- 原因：PostgreSQL 不支持 `DOUBLE` 数据类型

### 修复计划

- [x] 分析当前数据库配置类型
- [x] 修复 DOUBLE 类型为 PostgreSQL 兼容类型
- [x] 验证修复结果

### 执行步骤

1. **确认数据库配置**：

   - 查看了 `.env.dev` 文件，确认 `DB_TYPE = 'postgresql'`
   - 确认使用的是 PostgreSQL 数据库

2. **修复数据类型问题**：

   - 将 `from sqlalchemy import ... DOUBLE, ...` 改为 `from sqlalchemy import ... Float, ...`
   - 将 `ApschedulerJobs` 类中的 `next_run_time` 字段从 `DOUBLE` 改为 `Float`

3. **验证修复结果**：
   - 重新启动应用程序
   - 应用成功启动，没有出现 `type "double" does not exist` 错误
   - 数据库连接成功，Redis 连接成功
   - 定时任务系统正常启动
   - 应用在 http://0.0.0.0:9099 端口正常运行

### 修复总结

**问题根本原因**：

- PostgreSQL 数据库不支持 `DOUBLE` 数据类型
- SQLAlchemy 的 `DOUBLE` 类型在 PostgreSQL 中无法正确映射

**解决方案**：

- 将 SQLAlchemy 的 `DOUBLE` 类型改为 `Float` 类型
- `Float` 类型在 PostgreSQL 中会自动映射为 `DOUBLE PRECISION`，这是 PostgreSQL 的双精度浮点数类型

**修改的文件**：

- `ruoyi-fastapi-backend/module_admin/entity/do/job_do.py`
  - 导入语句：`DOUBLE` → `Float`
  - `ApschedulerJobs.next_run_time` 字段类型：`DOUBLE` → `Float`

**验证结果**：
✅ 应用启动成功
✅ 数据库表创建正常
✅ APScheduler 表创建成功
✅ 定时任务系统正常运行
