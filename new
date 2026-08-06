# 人工测试清单与现存问题详细说明

> 本文档由 `MD/system-features-business-processes-zh.md`(2026-08-06 静态分析报告)归纳整理,分为两部分:
> - **第一部分 需要人工测试的事项**:静态分析无法下结论、必须通过运行环境或外部服务实测才能验证的点。
> - **第二部分 目前存在的问题**:源码已能证明的缺陷、断点、未闭环能力,无需运行即可定性。
>
> 状态口径沿用原报告:`已闭环 / 部分闭环 / 未形成闭环 / 外部闭环未知 / 遗留·未使用 / 无法确认`。

---

## 第一部分 需要人工测试的事项

### 1. 外部服务依赖(均为"外部闭环未知",必须端到端实测)

| # | 外部依赖 | 业务用途 | 仓库内失败处理 | 最小人工验证方法 |
|---|---|---|---|---|
| E-01 | 微信 `jscode2session` | code 换 openid,登录前置 | 无 openid 时返回空 token/openid | 用测试账号覆盖成功、无效 code、超时、限流,核对返回分类 |
| E-02 | 微信 access token | 内容审核、手机号、小程序码共用 | 缓存未命中路径会 `exit`,首次/过期后可能中断业务 | **清空缓存后首次调用**,观察发布/评论/咨询/认证是否被中断 |
| E-03 | 微信 `msg_sec_check` | 商品/内容/评论/咨询/认证/店铺/评价文本审核 | 非成功统一返回业务失败 | 非生产环境验证通过、拒绝、超时、token 过期恢复 |
| E-04 | 微信手机号接口 | 授权登录/注册 | 非 0 错误要求重授权 | 验证首次授权、拒绝授权、手机号已占用、重复登录 |
| E-05 | 国内短信网关(互亿) | 国内验证码 + 业务通知 | 同步非成功返回 false | 用供应商测试号码核对请求、回执、重复、失败重试 |
| E-06 | 国际短信网关 | 非 +86 验证码/通知 | 检查返回 `SUCCESS` | 按配置支持国家做端到端送达与号码规范验证 |
| E-07 | SMTP / PHPMailer | 邮箱验证码、报价/审核/结拍通知 | 仅返回 `send()` 结果 | 测试成功、认证失败、退信、重试 |
| E-08 | WebSocket 服务 | 未读角标、广播、新报价事件 | 客户端 25 秒心跳、最多 10 次重连 | 检查部署进程、auth 协议、事件生产者、离线补发、多端顺序 |
| E-09 | 图片/对象存储 | 头像、业务图片、分享海报 | 同步上传/下载提示 | 核对存储映射、过期/删除、访问权限、容量、垃圾清理 |
| E-10 | 微信小程序码 | 独立脚本 `generate_qrcode.php` 生成 PNG | 错误时生成带错误文字的 PNG | 检查 Web 服务器是否暴露该脚本及业务调用方 |

遗留 helper(Aliyun SMS、IP 地理位置、百度翻译)当前无调用,搜索生产分支和调用日志确认是否真为死代码再决定保留或移除。

---

### 2. 静态分析无法确认清单(UNC-01 ~ UNC-16)

这些点源码层面有矛盾或读不出结论,**必须在隔离/测试环境实测**才能定性。

| ID | 问题 | 现有证据 / 不能确认原因 | 最小人工验证 |
|---|---|---|---|
| UNC-01 | API 域根路由 | `/ -> api/Home/index`,但 `Home` 无 `index` 方法 | 隔离环境请求 API 根,查看框架实际错误/兜底 |
| UNC-02 | 手机换绑目标 | `setMyPhone` 从假值 `$res` 读 `$res['id']` | 用测试会员 + 未占用号码换绑,写前后查 `ak_member.mobile` |
| UNC-03 | 联系人认证 guard | `if(!$minfo && $minfo['company_status']<2)` 逻辑错位 | 用未认证会员分别读/增/改/删联系人,核对对象范围 |
| UNC-04 | 提前结拍方向 | `delGoodsOrder` 用未定义 `$val['btype']` | 对 `btype=1`、`btype=2` 各准备多报价,提前结束核对赢家 |
| UNC-05 | 撤回后评价清理 | `delOrder` 用 `ak_evaluate.id=ordId`,但订单号写在 `orderid` | 建评价后撤回报价,核对目标及相邻评价是否被误删 |
| UNC-06 | 网站报价死代码 | `Orders::quot` 两个返回分支后才更新 `flag/current_price` | 网站报价后对比 `ak_orders.flag` 与 `ak_goods.current_price` |
| UNC-07 | 网站改密 | `Member::setpwd` 中 `$pass` 赋值被注释,且更新前返回 | 网站改密后用新旧密码分别登录测试 |
| UNC-08 | 缺失表 | 代码引用 `ak_seo/ak_cart/ak_pro/ak_pro_comt/ak_address/ak_order_info/ak_config_config`,DDL 无 | 生产只读 `SHOW TABLES` 与 50 表清单比较 |
| UNC-09 | 缺失列 | `ak_member.tags`;`ak_store.apptime/shop_sort/shop_type/company/ent_reg_num/qyzh`;`ak_orders.order_no/remark/nums/total/name/mobile/address/create_time` | 生产只读 `SHOW COLUMNS` 比较三张表 |
| UNC-10 | WSS 服务端 | 仓库仅客户端 + Workerman 注释 | 检查部署主机监听进程/源码,抓测试会话的 auth、角标、断线补发 |
| UNC-11 | 两个启动接口 | `app.js` 请求 `getLangByIp/getServerTime`,无路由无 handler | 隔离/生产只读请求,检查是否由仓库外 Web 服务器重写 |
| UNC-12 | 后台未初始化 `$map` | 多个列表把未初始化变量传 `where()` | 测试环境启 SQL 日志,核对列表实际 WHERE |
| UNC-13 | `select()/find()` 全列响应 | 多 action 未指定 `field()`,实际 JSON 键取决于部署表结构 | 对照测试响应和 `SHOW COLUMNS`,只保留契约字段 |
| UNC-14 | 静态资源可达性 | 模板直接/间接(`@import/url()`)引用资源 | 构建资源引用图,浏览器网络面板检查 |
| UNC-15 | 后台规则覆盖 | `ak_rules` 业务行不在仓库;无规则行时已登录管理员可能直入 action | 导出生产路径规则清单,与 156 个 action 做集合差异 + 逐角色测试 |
| UNC-16 | API 控制器目录内静态站 | `api/controller/puhualink.com` 无 PHP 调用证据 | 检查 Web 服务器 document root/alias |

---

### 3. 有限可用 / 需人工配合的业务场景(来自 §13.2)

这些能力主路径已实现,但试运行期间必须由人工核对结果:

| 业务 | 可用部分 | 必须人工核对 |
|---|---|---|
| 微信/验证码登录、通知、图片上传、内容审核 | 调用链完整 | 外部服务配置和实际交付 |
| 个人/企业认证、服务资质 | 提交和审核入口齐备 | 运营审核;**两套审核入口(小程序审核员 + 后台)须避免同时操作同一申请** |
| 商品发布、编辑、删除、重发 | 正常路径实现 | 对象权限、完整生命周期 |
| 报价、提前/定时结拍、评价 | 可试运行 | 当前价、赢家、通知、评价资格的人工核对 |
| 咨询与未读消息 | HTTP 读取 + 轮询可用 | 实时送达**不能承诺** |
| 反馈/举报 | 可收集 + 后台可查看 | 结果须人工另行通知提交人 |
| 关税/统计资料 | 页面查询可用 | 导入完整性和资料时效 |
| 公共网站展示与部分会员能力 | 辅助入口可用 | 写操作需与小程序数据人工核对 |

---

## 第二部分 目前存在的问题

### 1. 关键缺口总览(GAP-001 ~ GAP-026,来自 §10)

按业务影响排序。每项均附源码证据,无需运行即可定性。

#### A. 安全与权限类

| GAP | 问题 | 影响 | 证据 |
|---|---|---|---|
| **GAP-007** | 商品编辑/删除/提前结束**未验证商品归属** | 商家可管理他人商品 | `api/Good::setGoodsAddDatas/setGoodsDelDatas`、`api/Orders::delGoodsOrder` |
| **GAP-005** | 店铺联系人增删改查缺归属校验,且认证 guard `&&` 逻辑错位 | 联系人业务边界不能承诺 | `api/Store::getStoreLx/setStoreLx/setStoreLxDel` |
| **GAP-014** | 文章/动态编辑、删除按传入 id 操作,**未证明归属** | 内容治理边界不完整 | `api/Article.php`、`api/Dt.php`、`index/Index.php` |
| **GAP-018** | 生产 `ak_rules` 全覆盖缺失,无规则 action 的默认策略不明 | 不能证明不同管理员的实际操作边界 | `madmin/Base::checkLogin`、DDL `ak_rules/ak_group_rule` |
| **GAP-001** | JWT 无过期、无刷新、无服务端撤销;改密/注销后旧 token 仍可解码 | 无法承诺会话生命周期和跨端退出 | `api/common.php — createToken/checkToken` |

#### B. 交易一致性类(最关键)

| GAP | 问题 | 影响 | 证据 |
|---|---|---|---|
| **GAP-009** | 报价未检查商品 `status/order_status`、未检查卖家自报;读最佳价与写新价**无锁无事务** | 并发报价时不能保证唯一正确当前价 | `api/Good::setGoodsQuot` |
| **GAP-010** | 提前结拍用未定义 `$val['btype']`,**固定走升序** | 出售(btype=1)和求购(btype=2)方向可能选错赢家 | `api/Orders::delGoodsOrder` |
| **GAP-011** | `AutoTask` 先写 `order_status=1` 再通知;崩溃后**不重试**;无锁,多实例可竞争 | 中途失败留下"已结束但未通知/未完整选定" | `AutoTask::doWork` |
| **GAP-012** | 撤回报价后**不重算** `current_price`/最佳 `flag`/参与人数;评价清理用错主键 | 页面价格和参与状态陈旧;可能误删评价 | `api/Orders::delOrder` |
| **GAP-013** | 评价未验证赢家/成交/未评价资格;评分重算与评价写入无事务 | 评价终态与店铺评分不可信 | `api/Orders::setOrderPJ/getOrderPJDatas/setOrderPJReply/setOrderPJDel` |
| **GAP-023** | 站内信/邮件/短信与本地状态**非原子**;无持久发送任务、回执、重试 | 业务成功 ≠ 通知送达 | `addInfo/send_email/sendSMS/sendSingleIMS` |

#### C. 审核与状态机类

| GAP | 问题 | 影响 | 证据 |
|---|---|---|---|
| **GAP-004** | 认证审核有**两套独立写路径**(小程序 `setReviewOperat` + 后台 `Member::checkst`/`Dc::checkst`),无领取、版本号、防重复决定 | 两个审核人可覆盖彼此结果,通知与状态分离 | `api/Message::setReviewOperat`、`madmin/Member::checkst`、`madmin/Dc::checkst` |
| **GAP-006** | 服务选择 / 已批资质 / 取消恢复**无统一状态机** | 改店铺设置可能让已批服务无提示退出目录 | `api/Store::setStore`、DDL `ak_store_serve` |
| **GAP-008** | 商品生命周期无用户侧"下架/恢复/归档"状态机;发布后无运营审核队列 | 终态不可解释 | `api/Good.php`、`madmin/Goods.php`、DDL `ak_goods` |
| **GAP-015** | 举报附件 UI 有选择但 API 未给 `pic` 赋值;后台处理后**不通知提交人**;无申诉/关闭 | 提交人无法知道结果,运营证据链不完整 | `api/My::setMyFeedback`、`madmin/Feedback.php` |

#### D. 数据完整性类

| GAP | 问题 | 影响 | 证据 |
|---|---|---|---|
| **GAP-017** | 注销物理删除约 20 类关联表;无事务/续作/冷静期/归档/token 吊销;50 表 0 外键 | 中途失败出现**半注销且不可恢复** | `api/My::cancelAccount` |
| **GAP-024** | 50 张表 **0 外键**,MyISAM/InnoDB 混用,核心写跨引擎不可事务 | 删除/计数/竞拍/审核一致性完全依赖应用自觉 | DDL 全部 `CREATE TABLE` |
| **GAP-019** | 海关/关税表格导入**先清表再逐行写**,无批次/预校验/事务/回滚 | 坏文件或中途失败可清空或部分覆盖参考资料 | `madmin/Import::import`、`madmin/Tariff::import` |

#### E. 功能未闭环 / 死代码

| GAP | 问题 | 影响 | 证据 |
|---|---|---|---|
| **GAP-002** | `app.js` 启动调 `getLangByIp/getServerTime`,**无路由无 handler** | 启动初始化依赖不可承诺 | `auctionFish/app.js`、`route.php` |
| **GAP-003** | `setMyPhone` 更新分支从假值 `$res` 取 id | 用户可能无法完成手机换绑 | `api/My::setMyPhone` |
| **GAP-020** | 网站 `Orders::quot` 插入后即返回,聚合更新**不可达**;`Member::setpwd` 写库前提前返回 | 网站显示成功但关键状态未完成 | `index/Orders::quot`、`index/Member::setpwd` |
| **GAP-021** | 网站遗留购物车/订单/旧商家申请页引用 DDL 缺失表/列 | 不能作为交付范围承诺 | `index/Cart.php`、`index/Order.php`、`index/Member::merchantApply/memTags` |
| **GAP-022** | 161 条路由中 **5 个目标缺方法、2 个模式重复** | 链接可能错误或行为取决于路由覆盖顺序 | `tp/application/route.php` |
| **GAP-016** | WSS 客户端有,`join_goods` 无页面调用、`new_bid` 回调无页面注册,**服务端缺失** | 只能承诺刷新读取,不能承诺实时交付 | `utils/websocket.js` |
| **GAP-025** | 微信 `getAccessToken` 缓存未命中路径会 `exit` | 首次/过期后发布、评论、咨询、认证可能被中断 | `api/common.php — getAccessToken/WxTextVerification` |
| **GAP-026** | 后台有 `Orders` 两个 HTML 屏,但 `controller/Orders.php` **为空** | 客服无法形成订单/争议处理闭环 | `madmin/view/Orders/**`、空 `madmin/controller/Orders.php` |

---

### 2. 显式路由中的非功能断点(§5.5)

| 路由声明 | 问题 |
|---|---|
| `/ -> api/Home/index` | `Home::index` 方法不存在 |
| `fl -> Product/fl` | `Product::fl` 不存在 |
| `checkOrder -> Product/checkOrder` | `Product::checkOrder` 不存在 |
| `goodsType -> Store/goodsType` | `Store::goodsType` 不存在 |
| `goodsTypeDel -> Store/goodsTypeDel` | `Store::goodsTypeDel` 不存在 |
| `goods/:items` | 重复声明两次 |
| `search` | 重复声明两次 |

---

### 3. 功能级别状态统计(§5 汇总)

#### 3.1 小程序/API/任务 87 项核心能力状态分布

| 状态 | 数量 | 代表项 |
|---|---|---|
| 已闭环 | 约 35 | 浏览、搜索、详情、只读资料、参考表查询、系统消息、关注通知等边界清晰的能力 |
| 部分闭环 | 约 40 | 报价、结拍、评价、发布、删除、认证、服务资质、咨询等含权限/一致性/通知断点的链 |
| 外部闭环未知 | 约 8 | 微信登录、验证码、微信手机号、内容审核依赖外部交付的环节 |
| **未形成闭环** | 3 | `feat.member.phone.change`(手机换绑 id 错误)、`feat.mp.getServerTime`、`feat.mp.getLangByIp` |
| 遗留/未使用 | 1 | `feat.api.home.products`(无路由、无客户端入口) |

#### 3.2 管理后台 156 项 action

- 全部为 **部分闭环** 或更差,主因:RBAC 规则行数据不在仓库,授权覆盖**无法确认**。
- `Auth`、`Ert`、`Group`、`Index`、`Store` 等控制器因引用 DDL 缺失表/列或依赖生产规则数据,评为 **无法确认**。
- `Orders` 控制器为空文件,不构成后台订单能力。

#### 3.3 公共网站 86 项

- 9 个 widget 与部分 Index/Login 只读页为 **已闭环**。
- 大量会员/商家/评论/收藏/关注链为 **部分闭环**(与小程序规则重复且不一致)。
- **未形成闭环**:`Member::setpwd`(改密死代码)、`Product::shopInfo`(IP 条件后方法体为空)。
- **遗留/未使用**:`Cart` 4 项、`Order` 4 项(依赖 DDL 缺失表)。

---

### 4. 业务流程闭环评估(§8 矩阵汇总)

**结论:13 条核心业务流程中,0 条达到生产级"已闭环"。**

| 流程 | 整体结论 | 最薄弱环节 |
|---|---|---|
| FLOW-01 注册登录 | 部分闭环 | token 生命周期、跨端退出、外部发送重试 |
| FLOW-02 认证开店 | 部分闭环 | 双审核路径无防重复 |
| FLOW-03 店铺维护 | 部分闭环 | 联系人对象权限为 `—`(缺失) |
| FLOW-04 商品生命周期 | 部分闭环 | 编辑/删除对象归属为 `—` |
| **FLOW-05 竞拍成交评价** | **部分闭环(最关键)** | 并发恢复、跨表一致性均为 `—` |
| FLOW-06 服务资质 | 部分闭环 | 双审核路径、取消无独立闭环 |
| FLOW-07 内容与举报 | 部分闭环 | 编辑/删除归属、跨端审核规则不一致 |
| FLOW-08 咨询消息 | 部分闭环 | 实时推送为外部未知,终态为 `—` |
| FLOW-09 反馈举报 | 部分闭环 | 失败恢复、结果反馈、终态均为 `—` |
| FLOW-10 资料与注销 | 部分闭环 | 跨表一致性为 `—`,注销无事务 |
| FLOW-11 后台运营 | **无法确认** | 权限覆盖无法确认 |
| FLOW-12 外部/异步 | 外部闭环未知 | 全链依赖仓库外服务 |
| FLOW-13 网站重叠链 | 部分闭环 | 报价/改密死代码、跨端规则分叉 |

---

### 5. 多端规则不一致清单(§11,最易引发数据矛盾)

| 业务 | 小程序/API | 公共网站 | 差异风险 |
|---|---|---|---|
| 认证资格阈值 | `company_status>=2` | `company_status==2` | 同一会员两端判定不同 |
| 出价企业认证限制 | **被注释**(不要求) | 仍要求认证 | 资格判断分叉 |
| 报价聚合更新 | 正常更新 `flag/current_price` | **插入后即返回,更新不可达** | 跨端价格/报价状态不一致 |
| 改密 | `setInfo` 可更新,JWT 不失效 | `setpwd` 写库前返回,实际不改 | 网站改密无效 |
| 内容评论审核 | 调微信 `msg_sec_check` | 直接写入,不审核 | 同一内容两端适用不同规则 |
| 收藏/关注/粉丝数 | 各自独立写入 | 各自独立写入 | 计数无单一写入口,无 DB 约束 |
| 会话体系 | JWT,各 action 自行 `checkToken` | Session,`_initialize` 统一检查 | 三套会话互不联动,退出/改密/注销不跨端失效 |

---

## 附:优先级建议

按"影响面 × 出现概率"排序,建议优先处理:

1. **GAP-009 / GAP-010 / GAP-011**:竞拍并发、方向、结拍恢复——直接关系交易正确性。
2. **GAP-007 / GAP-005 / GAP-014**:对象归属校验——关系越权风险。
3. **GAP-004**:审核双路径统一——关系运营可追溯性。
4. **GAP-024 / GAP-017**:数据库约束与注销事务——关系数据完整性底线。
5. **GAP-001**:JWT 生命周期——关系会话安全。
6. 其余 GAP 与 UNC 项配合人工测试逐步收敛。

> 验证原则:UNC 项必须在**隔离测试环境**用真实测试账号实测,禁止凭静态推断下结论;涉及生产的只读核查(如 `SHOW TABLES/COLUMNS`)须以只读权限执行,不得以猜测补齐 DDL。
