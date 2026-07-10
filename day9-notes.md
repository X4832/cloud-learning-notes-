# 阿里云 LNMP+RDS+ALB 负载均衡部署 WordPress 

## 实训简介
基于阿里云公有云资源搭建三层 Web 业务架构：应用型负载均衡 ALB → ECS (Linux+Nginx+PHP-FPM) → RDS MySQL，完成 WordPress 个人博客完整部署，覆盖云服务器、云数据库、负载均衡全流程配置、页面故障排查与链路验收。
## 技术栈
Linux (CentOS)、Nginx、PHP-FPM、WordPress、阿里云 ECS、RDS MySQL、ALB 应用型负载均衡

# 📚 学习内容
- LNMP 架构组成与各组件作用
 -Nginx 网站根目录、PHP-FPM 运行权限原理
- RDS 云数据库概念、与自建 MySQL 差异对比
- RDS 内网白名单、账号、数据库创建流程
- WordPress 源码部署、wp-config.php数据库连接配置
- Linux 文件属主、文件权限原理（nginx/apache 用户权限踩坑）
- WordPress 站点初始化流程、安装状态校验逻辑
- 阿里云 ALB 负载均衡核心组件：实例、后端服务器组、监听、健康检查
- 三层业务访问链路：SLB公网IP → ECS Web → RDS
- WordPress 典型故障：循环跳转setup-config.php、无法写入配置文件、SLB 后台访问拦截

   |设备 / 服务	|工作层级	|寻址 / 访问依据	|  核心作用     |
   |------------|---------|-----------------|---------------|
   |Hub 集线器	  |物理层	  |无               |广播所有流量，无隔离|
   |Switch 交换机	|数据链路层	|MAC 地址|	根据 MAC 地址单播转发，隔离冲突域|
   |Router 路由器	|网络层|	IP 地址	|跨网段路由转发，隔离广播域|
   |Nginx Web 服务|应用层	|HTTP 请求	|静态资源分发、PHP 请求转发|   
   |RDS 云数据库	|应用层	|内网域名 + 账号密码|	托管 MySQL，持久化站点数据|
   |ALB 负载均衡	|应用层	|公网/IP	|流量分发、后端健康检查、统一业务入口|
   
## RDS 自建数据库 vs 云数据库对比
| **设备 / 服务** | **工作层级** | **寻址 / 访问依据**      | **核心作用**                                 |
|----------------|-------------|--------------------------|----------------------------------------------|
| Hub 集线器      | 物理层       | 无                       | 广播所有流量，无隔离                           |
| Switch 交换机   | 数据链路层   | MAC 地址                 | 根据 MAC 地址单播转发，隔离冲突域              |
| Router 路由器   | 网络层       | IP 地址                  | 跨网段路由转发，隔离广播域                     |
| Nginx Web 服务  | 应用层       | HTTP 请求                | 静态资源分发、PHP 请求转发                     |
| RDS 云数据库    | 应用层       | 内网域名 + 账号密码       | 托管 MySQL，持久化站点数据                     |
| ALB 负载均衡    | 应用层       | 公网IP                   | 流量分发、后端健康检查、统一业务入口            |

# 🎯 实验目标
- ECS 云服务器搭建完整 LNMP 运行环境，启动 Nginx、PHP-FPM；
- 阿里云控制台创建 RDS MySQL 实例，新建业务库wordpress、访问账号wpuser；
- 配置 RDS 内网白名单，使用 ECS 远程连接数据库，验证数据表创建；
- 部署 WordPress 中文源码，生成标准数据库配置文件，修复 Linux 文件权限故障；
- 完成 WordPress 站点初始化，创建管理员账号，ECS 公网 IP 正常访问博客；
-  部署 ALB 应用型负载均衡，绑定后端 ECS，配置 80 端口监听与健康检查；
- 使用 SLB 公网 IP 访问 WordPress，验证三层完整业务链路通畅；
- 排查并解决页面循环跳转setup-config.php、权限不匹配、SLB 后台访问拦截等故障；
- 归档全套实验截图，整理完整实操流程文档。

# 📝 分步实验操作流程
## 一、ECS 搭建 LNMP Web 基础环境
- SSH 远程连接阿里云 CentOS ECS 实例，更新系统依赖包；
- 离线 / 在线安装 Nginx、PHP-FPM、php-mysqlnd 数据库驱动；
- 修改 Nginx 默认站点根目录为 /usr/share/nginx/html；
- 设置 Nginx、php-fpm 开机自启，启动服务并验证端口监听正常；
- 确认 Web 运行进程用户组为nginx，为后续权限配置做铺垫。
## 二、阿里云 RDS 云数据库配置实验
- 进入阿里云 RDS 控制台，与 ECS同地域创建 MySQL 实例；
- 实例创建完成后，新建业务数据库：CREATE DATABASE wordpress;
- 创建专用数据库访问账号wpuser，设置登录密码FXY20060207fxy；
- 将 ECS 内网 IP 添加至 RDS 白名单，仅允许 ECS 内网访问数据库；
- ECS 内执行 MySQL 连接命令测试连通性：
- bash
- 运行
mysql -h rm-bp1z7x2n3e8j1altv.mysql.rds.aliyuncs.com -u wpuser -p wordpress

- 执行show tables;，确认 WordPress 初始化后可自动生成全套wp_*业务数据表。
## 三、WordPress 源码部署与数据库配置
- ECS 内下载 WordPress 中文源码压缩包，解压至 Nginx 网站根目录；
- 复制官方配置模板wp-config-sample.php生成业务配置文件wp-config.php；
- 替换数据库连接参数，填入 RDS 实例信息：
- php
- 运行
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wpuser' );
define( 'DB_PASSWORD', 'FXY20060207fxy' );
define( 'DB_HOST', 'rm-bp1z7x2n3e8j1altv.mysql.rds.aliyuncs.com' );
define( 'DB_CHARSET', 'utf8mb4' );
define( 'DB_COLLATE', '' );

- 调用 WordPress 官方密钥接口生成安全随机密钥，替换配置内默认密钥段；
- 核心权限修复操作（解决页面跳转故障）：
bash
- 运行

## 标准修复命令
chown -R nginx:nginx /usr/share/nginx/html
chmod 644 /usr/share/nginx/html/wp-config.php
systemctl restart php-fpm

## 四、WordPress 站点初始化安装
- 浏览器访问 ECS 公网 IP，进入 WordPress 安装向导；
- 填入 RDS 数据库信息，程序自动向 RDS 库写入 12 张 wp 业务数据表；
- 创建站点管理员账号：用户名x4832，绑定邮箱3389594117@qq.com；
- 初始化完成，验证 ECS 公网 IP 可正常打开博客前台首页，截图归档。
## 五、阿里云 ALB 负载均衡三层链路部署
5.1 创建 ALB 负载均衡实例
- 阿里云控制台搜索「负载均衡 SLB」，切换至 ECS 相同地域；
- 选择应用型负载均衡 ALB，按量付费、分配公网 IP、按流量计费；
- 实例命名：WP 实训负载均衡，等待实例创建完成。
5.2 后端服务器组配置
- 新建服务器组，类型为「ECS 服务器实例」；
- 将当前 Web ECS 添加至后端组，后端转发端口 80，权重默认 100；
- 配置 HTTP 健康检查：访问路径/，检测间隔 5s，超时 2s。
5.3 前端 80 端口监听配置
- 添加 HTTP 监听，前端端口 80，转发目标为后端 ECS 服务器组；
- 调度算法选择加权轮询，关闭会话保持功能。
5.4 ECS 安全组放行 SLB 访问流量
- 进入 ECS 实例安全组配置页面；
- 添加入站规则，授权 SLB 内网网段访问 80 端口，保证健康检查正常。
5.5 获取 SLB 公网访问 IP
- 复制实例详情页公网 IP，作为业务统一访问入口。

## 六、三层链路验收测试（实验验收标准）
- 浏览器无痕窗口输入 SLB 公网 IP，正常加载x4832的个人博客首页；
- 从首页底部「登录」入口进入后台，禁止直接手动访问/wp-admin路径；
- 验证完整业务链路：ALB公网入口 → Nginx Web服务 → RDS云数据库 全链路无报错；

## 🛠️ 故障排查记录（实验踩坑总结）
- 故障 1：访问任意页面强制跳转 setup-config.php
- 现象：首页、后台地址全部跳转配置页面，提示wp-config.php 已经存在
- 根因：网站文件属主被错误修改为apache，Nginx+PHP-FPM 运行用户为nginx，PHP 无法读取配置文件，WordPress 判定站点未完成初始化；
- 解决方案：一键修正全站文件归属，重启 php-fpm 刷新缓存；若无效则删除wp-config.php重新走完完整安装流程。
- 故障 2：网页提示「无法写入 wp-config.php 文件」
- 根因：网站目录权限归属非 Web 运行用户，WordPress 无文件写入权限；
- 解决方案：统一全站文件归属nginx:nginx，配置文件权限锁定为 644。
- 故障 3：SLB 公网 IP 访问/wp-admin直接跳转配置页
- 根因：手动直接访问后台路由，WordPress 安装状态校验异常触发拦截；
- 解决方案：从博客首页底部登录入口进入后台，不直接输入后台路径访问。

# ✅ 学习成果
- [x] 理解交换机、VLAN、MAC 地址表基础网络原理；
- [x] 掌握 RDS 云数据库与自建 MySQL 的差异、托管数据库优势；
- [x] 掌握 LNMP 架构组件分工、Linux 文件权限与 Web 进程用户匹配逻辑；
- [x] 独立完成 RDS 实例创建、内网白名单、远程数据库连接配置；
- [x] 熟练部署 WordPress 站点，独立编写数据库配置文件；
- [x] 能定位并解决 WordPress 页面循环跳转、文件权限类故障；
- [x] 掌握阿里云 ALB 负载均衡完整配置流程：实例、后端组、监听、健康检查；
- [x] 可独立完成三层 Web 业务链路部署、全链路连通性验收测试。
