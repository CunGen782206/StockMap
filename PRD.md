你现在是我的全栈开发代理，请直接在当前仓库中完成一个“股票产业链地图软件”MVP，要求前端使用 Vue 3 + TypeScript + Vite + Pinia + Vue Router + Element Plus，后端使用 C# ASP.NET Core Web API（controller-based）+ EF Core + SQLite（默认开发库）+ 可切换 SQL Server。请不要只输出方案，而是直接创建和修改代码，直到项目可以本地运行。

========================
一、项目目标
========================
项目名称：StockMap

项目定位：
这是一个“股票产业链地图 + 事件驱动分析”的软件，不是普通行情软件。
第一版聚焦 AI 算力产业链，提供：
1. 股票列表与搜索
2. 产业链地图（板块 -> 子板块 -> 股票）
3. 个股详情页
4. 事件列表与事件详情页
5. 自选股管理
6. 管理后台最简版：维护股票、板块、事件、关系
7. 首页仪表盘：热门板块、近期事件、自选股动态摘要

MVP 只做“研究工具”，不接券商交易，不做复杂实时行情，不做支付。
MVP 数据先以“种子数据 + 后台维护”为主，保留后续接行情源和新闻源的扩展能力。

========================
二、技术栈与硬性要求
========================
前端：
- Vue 3
- TypeScript
- Vite
- Pinia
- Vue Router
- Element Plus
- Axios
- ECharts（首页图表）
- Cytoscape.js 或者 vue-flow（二选一，优先 Cytoscape.js）用于产业链关系图
- Composition API
- 页面风格：简洁、专业、偏投研工具风格

后端：
- .NET 10
- ASP.NET Core Web API（Controllers）
- EF Core
- SQLite（默认）
- 可切换 SQL Server
- Swagger / OpenAPI
- AutoMapper
- FluentValidation（如工作量过大可用手写校验替代）
- JWT 鉴权（管理员登录）
- 分层结构：Controllers / Services / Repositories / Entities / DTOs / Infrastructure
- 所有接口返回统一响应结构

工程要求：
- 前后端分离
- 后端开启 CORS，支持本地前端开发
- 提供 README，写清楚启动方式
- 代码应可直接运行，不要留下大量 TODO 占位
- 不要只创建骨架，要尽量实现完整可用功能
- 优先保证可运行和结构清晰，其次再做花哨效果

========================
三、核心业务域设计
========================
1. 股票 Stock
字段：
- Id
- Ticker（如 NVDA / 300394.SZ）
- Name
- Market（A股 / 美股 / 港股）
- Sector（一级行业）
- Industry（二级行业）
- Description
- Country
- Currency
- IsActive
- CreatedAt
- UpdatedAt

2. 主题/板块 Theme
字段：
- Id
- Name（如 AI算力 / HBM / 光模块 / CPO / AIDC）
- Category（主线 / 子板块 / 概念）
- Description
- ParentThemeId（可空，支持树形结构）
- SortOrder
- IsActive
- CreatedAt
- UpdatedAt

3. 股票-主题关系 StockThemeRelation
字段：
- Id
- StockId
- ThemeId
- RelationType（核心 / 次核心 / 映射 / 观察）
- Weight（0-100）
- Note

4. 事件 Event
字段：
- Id
- Title
- EventType（财报 / 产品发布 / 政策 / 涨价 / 订单 / 资本开支 / 机构观点 / 其他）
- Source
- Summary
- Content
- EventDate
- ImpactLevel（低 / 中 / 高）
- IsActive
- CreatedAt
- UpdatedAt

5. 事件-股票关系 EventStockRelation
字段：
- Id
- EventId
- StockId
- RelationType（直接受益 / 间接受益 / 风险 / 情绪映射）
- ImpactScore（0-100）
- Note

6. 股票关系 StockRelation
字段：
- Id
- SourceStockId
- TargetStockId
- RelationType（上游 / 下游 / 对标 / 替代 / 竞争 / 同涨逻辑）
- Strength（0-100）
- Note

7. 自选股 Watchlist
字段：
- Id
- UserId
- Name
- Description
- CreatedAt

8. 自选股明细 WatchlistItem
字段：
- Id
- WatchlistId
- StockId
- Tag（核心龙头 / 观察中 / 等回调 / 事件博弈）
- Note
- CreatedAt

9. 用户 User
字段：
- Id
- UserName
- PasswordHash
- Role（Admin / User）
- DisplayName
- CreatedAt
- UpdatedAt

========================
四、默认种子数据要求
========================
请初始化一批与 AI 算力链相关的演示数据，至少包含以下主题：
- AI算力
- GPU
- HBM
- 光模块
- 光芯片
- CPO
- PCB/交换机
- 服务器电源
- 液冷
- AIDC

至少包含以下股票示例（可补充）：
- NVDA
- AMD
- MU
- AVGO
- LITE
- COHR
- 中际旭创
- 新易盛
- 天孚通信
- 光迅科技
- 润泽科技
- 光环新网
- 工业富联
- 通富微电

并建立合理关系：
- 股票 -> 主题
- 股票 -> 股票
- 事件 -> 股票

还要创建至少 8 条演示事件，例如：
- 英伟达发布新一代 AI GPU
- Micron HBM 进展更新
- CPO 渗透率预期提升
- AIDC 建设资本开支增加
- 某龙头公司财报超预期
等

========================
五、后端接口设计要求
========================
统一返回格式：
{
  "code": 0,
  "message": "success",
  "data": ...
}

错误时：
{
  "code": 非0,
  "message": "错误信息",
  "data": null
}

请实现以下 API：

1. 认证
- POST /api/auth/login
- GET /api/auth/me

2. 首页
- GET /api/dashboard/summary
返回：
{
  hotThemes: [],
  recentEvents: [],
  watchlistHighlights: [],
  stats: {
    stockCount,
    themeCount,
    eventCount
  }
}

3. 股票
- GET /api/stocks
  支持分页、关键字搜索、按市场筛选、按主题筛选
- GET /api/stocks/{id}
  返回股票详情、关联主题、关联事件、关联股票
- POST /api/stocks
- PUT /api/stocks/{id}
- DELETE /api/stocks/{id}

4. 主题
- GET /api/themes/tree
- GET /api/themes
- GET /api/themes/{id}
- POST /api/themes
- PUT /api/themes/{id}
- DELETE /api/themes/{id}

5. 股票-主题关系
- POST /api/stock-theme-relations
- DELETE /api/stock-theme-relations/{id}

6. 事件
- GET /api/events
  支持分页、按类型筛选、按日期排序
- GET /api/events/{id}
  返回事件详情及关联股票
- POST /api/events
- PUT /api/events/{id}
- DELETE /api/events/{id}

7. 事件-股票关系
- POST /api/event-stock-relations
- DELETE /api/event-stock-relations/{id}

8. 股票关系
- GET /api/stocks/{id}/relations
- POST /api/stock-relations
- DELETE /api/stock-relations/{id}

9. 自选股
- GET /api/watchlists
- POST /api/watchlists
- PUT /api/watchlists/{id}
- DELETE /api/watchlists/{id}
- GET /api/watchlists/{id}/items
- POST /api/watchlists/{id}/items
- DELETE /api/watchlists/items/{itemId}

10. 管理后台附加接口
- GET /api/admin/seed-status
- POST /api/admin/seed-reset

要求：
- Swagger 中可直接调试
- DTO 与 Entity 分离
- 使用 AutoMapper
- 需要基本输入校验
- 管理写操作默认要求 JWT Admin 权限
- 读接口对匿名用户开放，自选股接口可先默认单用户模式；如果你觉得用户体系完整实现成本过高，可先做一个默认测试用户 + 登录

========================
六、前端页面设计要求
========================
请实现以下页面和路由：

1. /login
- 管理员登录页
- 登录成功后保存 token

2. /
首页 Dashboard
内容：
- 顶部统计卡片：股票数 / 主题数 / 事件数
- 热门主题卡片
- 最近事件表格
- 自选股摘要卡片
- 一个简单图表（如事件数量趋势 / 市场分布）

3. /stocks
股票列表页
功能：
- 搜索框
- 市场筛选
- 主题筛选
- 表格展示
- 点击进入详情页

4. /stocks/:id
股票详情页
模块：
- 基本信息
- 所属主题标签
- 关联事件时间线
- 关联股票关系表
- 产业链位置图（局部图谱）
- 加入自选股按钮

5. /themes
主题树 / 产业链地图页
功能：
- 左侧主题树
- 右侧图谱区域
- 点击主题显示其下股票
- 支持查看父子主题与核心股票
- 图谱节点样式清晰：主题、股票、事件可区分

6. /events
事件列表页
功能：
- 按类型筛选
- 时间排序
- 表格/卡片切换二选一即可
- 点击进入详情

7. /events/:id
事件详情页
模块：
- 标题、类型、日期、来源
- 摘要、正文
- 关联股票列表
- 影响方向标签（直接受益/间接受益/风险）

8. /watchlists
自选股页
功能：
- 创建自选分组
- 添加股票
- 删除股票
- 标签和备注
- 简单表格展示

9. /admin
后台管理首页
10. /admin/stocks
11. /admin/themes
12. /admin/events
13. /admin/relations

后台要求：
- 用 Element Plus 的表单、表格、弹窗快速实现 CRUD
- 管理后台不追求特别复杂，但要可用

========================
七、前端工程结构要求
========================
请按合理方式组织，例如：

frontend/
  src/
    api/
    assets/
    components/
    composables/
    layout/
    router/
    stores/
    types/
    utils/
    views/
      auth/
      dashboard/
      stocks/
      themes/
      events/
      watchlists/
      admin/

要求：
- api 层统一封装 axios
- 自动携带 token
- 统一错误处理
- 路由守卫控制后台页面访问
- 类型定义尽量完整
- 页面使用 Composition API + script setup
- 尽量拆分通用组件，如：
  - ThemeTree.vue
  - StockRelationGraph.vue
  - EventTimeline.vue
  - PageHeader.vue
  - StatCard.vue

========================
八、后端工程结构要求
========================
请按合理方式组织，例如：

backend/
  StockMap.Api/
    Controllers/
    Services/
    Repositories/
    Entities/
    DTOs/
    Mappings/
    Data/
    Infrastructure/
    Validators/
    Common/

要求：
- Program.cs 中完成依赖注入
- EF Core DbContext
- Repository + Service 模式
- JWT 配置写入 appsettings.json
- CORS 允许前端本地地址
- 自动迁移或首次创建数据库
- 提供 SeedData 初始化逻辑
- 加上全局异常处理中间件或过滤器
- 支持 SQLite，保留 SQL Server 切换配置示例

========================
九、UI 与交互要求
========================
整体风格：
- 偏专业投研工具
- 浅色主题即可
- 左侧菜单 + 顶部 Header 的后台布局
- 首页信息密度中等，不要太花
- 图谱优先能用、清晰，不必追求炫酷动画

Element Plus 组件建议：
- ElContainer / ElAside / ElHeader / ElMain
- ElMenu
- ElCard
- ElTable
- ElForm
- ElDialog
- ElDrawer
- ElTree
- ElTag
- ElDescriptions
- ElTimeline
- ElPagination
- ElMessage

========================
十、实现优先级
========================
请严格按以下顺序推进，不要一开始就写一堆无关代码：

第 1 步：初始化前后端工程与目录结构
第 2 步：完成后端实体、DbContext、迁移、种子数据
第 3 步：完成后端核心读接口（dashboard、stocks、themes、events）
第 4 步：完成前端路由、布局、API 封装、列表页和详情页
第 5 步：完成后台 CRUD
第 6 步：完成登录与 JWT
第 7 步：完善 watchlist、自定义标签、关系图谱
第 8 步：补 README、启动命令、测试说明

========================
十一、验收标准
========================
完成后必须满足：

1. 前端能启动
2. 后端能启动
3. 数据库能自动初始化并带种子数据
4. Swagger 可访问
5. 首页、股票页、主题图谱页、事件页、自选页都可正常打开
6. 股票详情页能看到：
   - 基本信息
   - 关联主题
   - 关联事件
   - 关联股票
7. 主题页能显示主题树和至少一个可用的关系图
8. 管理员能登录后台并对股票/主题/事件做基本 CRUD
9. README 清晰写明：
   - 如何启动后端
   - 如何启动前端
   - 默认管理员账号密码
   - 默认数据库位置
10. 如果实现过程中发现某些功能复杂度超出预期，优先保证主链路可用，并在 README 中说明简化点，但不要把核心页面留空

========================
十二、默认账号与运行要求
========================
请创建默认管理员：
- 用户名：admin
- 密码：admin123

本地运行预期：
后端：
- dotnet restore
- dotnet ef database update
- dotnet run

前端：
- npm install
- npm run dev

请在 README 中明确前端和后端默认端口。
前端需要在开发环境中把 API 基础地址配置为可调整的 env 变量。

========================
十三、代码风格要求
========================
- 后端命名规范清晰，DTO/Entity 分开
- 前端组件尽量不要写成超大单文件，适度拆分
- 不要引入明显多余的复杂架构
- 注释写在关键处，不要满屏废话
- 保持代码尽量可读、可维护

========================
十四、你现在就开始执行
========================
请不要只回复计划。
直接开始：
1. 初始化项目结构
2. 创建前后端工程
3. 编写代码
4. 直到项目具备可运行的 MVP
5. 最后给出变更摘要、运行方式、已完成清单、未完成但已知的简化点


执行补充规则：

1. 不要只生成项目骨架，必须尽量生成可运行代码。
2. 每完成一个阶段，就继续推进下一个阶段，不要停在“我已经为你创建了结构”。
3. 如果某个库集成困难，优先保证功能落地，不要因为追求复杂最佳实践而卡住主流程。
4. 前端先把核心页面跑通，再优化样式。
5. 后端先把读接口、种子数据、Swagger 跑通，再补写接口。
6. 图谱页至少做到“主题 + 股票”的可视化关系展示。
7. 自选股先按单用户模式实现即可。
8. 如果 SQL Server 配置较繁琐，先保证 SQLite 可运行，同时保留 SQL Server 配置示例。
9. 所有关键功能完成后，补 README。
10. 最终输出时，必须告诉我：
   - 创建了哪些目录和文件
   - 关键依赖有哪些
   - 默认端口
   - 默认管理员账号密码
   - 如何运行
   - 哪些地方做了简化

收尾要求：

1. 检查前端是否存在明显类型错误。
2. 检查后端依赖注入是否完整。
3. 检查数据库初始化和种子数据逻辑是否能工作。
4. 检查 Swagger 能否看到所有核心接口。
5. 检查前端页面是否至少能完成：
   - 首页展示
   - 股票列表与详情
   - 主题树与关系图
   - 事件列表与详情
   - 自选股 CRUD
   - 管理后台基础 CRUD
6. README 必须足够让我本地手动启动成功。
7. 最终总结要按“已完成 / 简化点 / 运行步骤 / 下一步建议”输出。

本次只做 MVP，不做以下内容：
1. 不接真实券商交易
2. 不做复杂实时行情推送
3. 不做支付系统
4. 不做多租户
5. 不做复杂权限模型
6. 不做移动端适配优先
7. 不做 SSR
8. 不做大模型接入，AI分析先留扩展点


数据库要求（强制）：

1. 本项目数据库必须使用 SQL Server，不允许使用 SQLite 作为默认数据库。
2. 使用 EF Core + SQL Server Provider。
3. 默认连接字符串写在 appsettings.json 中，例如：

"ConnectionStrings": {
  "DefaultConnection": "Server=.;Database=DataBase;Trusted_Connection=True;TrustServerCertificate=True"
}

4. DbContext 必须配置为：
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

5. 必须使用 EF Core Migration：
- 初始迁移：InitialCreate
- 自动创建数据库
- 程序启动时自动应用迁移（Database.Migrate）

6. 必须提供完整步骤：
- dotnet ef migrations add InitialCreate
- dotnet ef database update

7. 所有字段必须兼容 SQL Server：
- string -> NVARCHAR
- DateTime -> datetime2
- bool -> bit
- decimal -> decimal(18,2)
- 禁止使用 SQLite 专有类型

8. 必须创建索引：
- Stock.Ticker（唯一）
- Theme.Name
- Event.EventDate
- StockThemeRelation (StockId + ThemeId)
- EventStockRelation (EventId + StockId)

9. 必须考虑未来数据量：
- 所有关联表必须有索引
- 主键使用 int identity

10. SeedData 必须适配 SQL Server，确保第一次启动可插入数据

11. 允许在 README 中写：
如何用 SQL Server Management Studio (SSMS) 查看数据


数据库执行规则补充：

1. 不允许回退到 SQLite。
2. 如果本地没有 SQL Server，请假设用户已安装 SQL Server Express 或 Developer。
3. 所有 Repository 和查询必须兼容 SQL Server。
4. 如果遇到数据库连接问题，优先修复连接字符串，而不是切换数据库类型。
5. 所有查询必须使用 LINQ，不允许写原生 SQL（除非必要）。


EF Core 要求：

1. 禁止在查询中使用会导致 client-side evaluation 的写法
2. 所有 Include / ThenInclude 必须明确
3. 分页必须使用 Skip + Take
4. 默认查询必须使用 AsNoTracking（只读接口）
5. 避免 N+1 查询问题
