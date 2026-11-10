# 课栈 CourseStack

> 把十几门课的截止日期、课件和待办收进一个地方。
> 一个只为自己写的课程管理系统，也是我把 Java 后端从语法练到能上线的试验田。

---

## 为什么做这个

大二上学期，我一共有 **17 门课**。每门课的信息分散在至少四个地方：

- 班级群里老师临时发的通知
- 课程群里的作业要求
- 雨课堂 / 学习通里的截止时间
- 教务系统里的调课、考试安排

结果是：**我每周大概要花 3 个小时，只是确认“有没有漏掉什么”。** 这 3 小时不能产出任何东西，而且随时会被打断。

这个项目解决的就是这件事——不是“更漂亮的待办清单”，是**把散落的课程信息收敛到一个地方，并且自动提醒**。

顺带，它也是我的第一个完整后端项目：从需求到上线，把 Spring Boot 的每个环节都走一遍。

---

## 功能范围（MVP）

| 模块 | 功能 | 状态 |
|---|---|---|
| 用户 | 注册 / 登录（JWT） | 计划中 |
| 课程 | 增删改查、按学期筛选、按周排课 | 计划中 |
| 截止日期 | 作业 / 报告 / 考试，按倒计时排序 | 计划中 |
| 课件 | 上传、按课程分组、在线预览 | 计划中 |
| 提醒 | 每日定时扫描，临期提醒 | 计划中 |
| 搜索 | 全局关键词搜索（课程 / 作业 / 文件名） | 计划中 |

**明确不做**：多人协作、消息推送 App、日历同步。这些会让项目失控。

---

## 技术栈

| 层 | 选型 |
|---|---|
| 语言 / 框架 | Java 17 + Spring Boot 3 |
| 数据库 | MySQL 8 + MyBatis-Plus |
| 认证 | JWT + Spring Security（简化配置） |
| 文件存储 | 本地磁盘（抽象成接口，后期可换 MinIO） |
| 定时任务 | Spring Scheduler |
| 接口文档 | SpringDoc OpenAPI（Swagger UI） |
| 测试 | JUnit 5 + H2 |
| 部署 | Docker + Nginx + 云服务器 |

---

## 数据模型

```
user       id, username, password_hash, email, created_at
course     id, user_id, name, teacher, semester, weekday, start_period, end_period, color
deadline   id, course_id, title, type(作业/报告/考试), due_at, status(未交/已交), note
material   id, course_id, filename, stored_path, size, uploaded_at
reminder   id, deadline_id, remind_at, channel(站内/邮件), sent
```

设计原则：**所有业务表带 `user_id` 或通过 `course_id` 关联到用户**，为将来多用户留口子，但现在只做单用户。

---

## 核心接口

| 方法 | 路径 | 说明 |
|---|---|---|
| POST | `/api/auth/register` | 注册 |
| POST | `/api/auth/login` | 登录，返回 JWT |
| GET | `/api/courses` | 课程列表（分页 + 学期筛选） |
| POST | `/api/courses` | 新增课程 |
| PUT | `/api/courses/{id}` | 修改课程 |
| DELETE | `/api/courses/{id}` | 删除课程 |
| GET | `/api/deadlines/upcoming` | 近 7 天截止（倒计时排序） |
| POST | `/api/materials/upload` | 上传课件 |
| GET | `/api/search?q=` | 全局搜索 |

统一返回体：

```json
{ "code": 0, "message": "ok", "data": {} }
```

错误码集中在 `ErrorCode` 枚举里，不散落魔法数字。

---

## 本地运行

> 开发中，跑起来之后补完整步骤。

```bash
git clone https://github.com/<你的用户名>/coursestack.git
cd coursestack
cp application-example.yml application.yml   # 改数据库密码
mvn spring-boot:run
```

访问 `http://localhost:8080/swagger-ui.html` 看接口文档。

---

## 开发里程碑

- [x] 需求梳理（就是这份 README）
- [ ] 项目骨架：统一返回体 + 全局异常 + 日志
- [ ] 用户模块：注册 / 登录 / JWT 校验
- [ ] 课程模块：CRUD + 分页 + 条件查询
- [ ] 截止日期模块：倒计时列表 + 状态流转
- [ ] 课件上传：本地存储 + 下载 + 预览
- [ ] 定时提醒：每日 8 点扫描，临期标记
- [ ] 接口文档 + 单元测试
- [ ] Docker 打包 + 服务器部署
- [ ] 写一篇开发复盘（踩了哪些坑）

---

## 设计取舍（面试会被问的）

**1. 为什么不用 JPA，用 MyBatis-Plus？**
项目里查询以后台管理式的条件组合为主（多条件筛选 + 分页），手写 SQL 更可控，也顺便把 SQL 练熟。代价是要自己维护 SQL，收益是能看懂执行计划。

**2. 为什么 JWT 不做 Redis 黑名单？**
单用户系统，token 有效期设为 7 天，登出只清前端。上生产再补 Redis + 短令牌 + 刷新机制——**这是一个明确的技术债，不是没想到**。

**3. 为什么文件存本地不直接上对象存储？**
先把链路跑通。`StorageService` 抽成接口，本地实现和 MinIO 实现可以互换，换的时候只改一个 Bean。

**4. 为什么定时任务用 Spring Scheduler 而不是 Quartz？**
任务只有一个：每天扫一遍截止日期。Quartz 带来的集群、持久化、补偿能力在这个规模下全是负担。

---

## 截图

> 开发完成后补：课程列表、倒计时视图、上传页面。

---

## 说明

这是我大二阶段的练手项目，代码质量和架构都在边做边改。如果你看到明显的问题，欢迎提 issue——比夸我更有用。
