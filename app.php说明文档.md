# app.php 核心功能库说明文档

## 文件概述

`app.php` 是 ScutChat 项目的核心功能库文件，包含了项目运行所需的所有常量定义、工具函数和初始化配置。该文件被 `index.php` 和 `ajax.php` 引入使用。

## 文件位置

```
app/app.php
```

## 代码结构

### 1. 错误报告设置

```php
error_reporting(0);
```

- **功能**：关闭所有错误报告
- **说明**：生产环境设置，避免错误信息泄露给用户

### 2. 会话管理初始化

```php
session_start();
```

- **功能**：启动PHP会话
- **说明**：用于存储用户登录状态、Token等信息

### 3. 路径常量定义

```php
define('BASE_PATH', str_replace('\\','/',dirname(__FILE__))."/");
define('ROOT_PATH', str_replace('app/','',BASE_PATH));
```

- **BASE_PATH**：应用目录的绝对路径（`app/` 目录）
  - 示例：`D:/360MoveData/Users/向/Desktop/scutchat/app/`
- **ROOT_PATH**：项目根目录的绝对路径
  - 示例：`D:/360MoveData/Users/向/Desktop/scutchat/`
- **说明**：使用 `str_replace` 将Windows路径分隔符 `\` 替换为 `/`，确保跨平台兼容性

### 4. 文件路径常量定义

```php
define('MSGFILE','app/MSG.txt'); 
define('NUMFILE','app/NUM.txt');
```

- **MSGFILE**：聊天消息存储文件的相对路径
- **NUMFILE**：数字统计文件的相对路径（当前未使用）
- **说明**：相对于项目根目录的路径

### 5. 安全密钥常量定义

```php
define('KEYS','bskdl87'); 
define('LSTR','admin');
```

- **KEYS**：Cookie密钥前缀
  - 用途：用于生成Cookie名称，如 `bskdl87_key`、`bskdl87_name`
  - 作用：防止Cookie名称冲突，增加安全性
- **LSTR**：管理员登录密钥
  - 用途：通过URL参数验证管理员身份
  - 使用：在登录时检查 `HTTP_REFERER` 的query参数

### 6. 时区设置

```php
date_default_timezone_set('PRC');
```

- **功能**：设置默认时区为中国时区（PRC = People's Republic of China）
- **说明**：确保时间显示正确，使用北京时间

## 核心函数说明

### 1. `nick($user = '')` - 用户昵称生成函数

#### 函数签名
```php
function nick($user='')
```

#### 功能描述
生成或使用指定的用户昵称，创建用户会话，并记录用户加入聊天的系统消息。

#### 参数说明
- `$user`（可选）：用户指定的昵称，如果为空则自动生成随机昵称

#### 返回值
返回关联数组：
```php
array(
    'name' => '生成的昵称',
    'key'  => '唯一标识符'
)
```

#### 工作流程
1. 如果 `$user` 为空，调用 `rand_nick()` 生成随机昵称
2. 如果 `$user` 不为空，使用用户指定的昵称
3. 生成两条系统消息：
   - 第一条：当前时间戳
   - 第二条：用户加入聊天的提示消息
4. 将系统消息追加写入 `MSG.txt` 文件
5. 生成唯一标识符 `key`（使用 `uniqid()`）
6. 设置Cookie：
   - `bskdl87_key`：用户唯一标识，有效期90天
   - `bskdl87_name`：用户昵称（URL编码），有效期30天
7. 返回昵称和key

#### 使用示例
```php
// 自动生成昵称
$userInfo = nick();
// 结果：array('name' => '随机昵称', 'key' => '唯一key')

// 使用指定昵称
$userInfo = nick('张三');
// 结果：array('name' => '张三', 'key' => '唯一key')
```

#### 注意事项
- Cookie使用 `KEYS` 常量作为前缀
- 昵称会进行URL编码存储
- 系统消息会立即写入文件

---

### 2. `rand_nick()` - 随机昵称生成函数

#### 函数签名
```php
function rand_nick()
```

#### 功能描述
生成随机昵称（当前实现不完整）。

#### 返回值
返回随机生成的昵称字符串。

#### 代码问题
```php
function rand_nick(){
  $t=rand(0,199);
  $w=rand(0,199);
  $name=$name_tou[$t].$name_wei[$w];
  return $name;
}
```

**注意**：此函数存在未定义变量的问题：
- `$name_tou` 和 `$name_wei` 数组未定义
- 当前实现会导致生成空昵称或报错

#### 建议修复
```php
function rand_nick(){
  $name_tou = array('小', '大', '老', '新', '好', ...); // 定义前缀数组
  $name_wei = array('明', '华', '强', '伟', '芳', ...); // 定义后缀数组
  $t = rand(0, count($name_tou)-1);
  $w = rand(0, count($name_wei)-1);
  $name = $name_tou[$t] . $name_wei[$w];
  return $name;
}
```

---

### 3. `logmsg($b, $msg = '成功！')` - 日志消息输出函数

#### 函数签名
```php
function logmsg($b, $msg = '成功！')
```

#### 功能描述
根据操作结果输出JSON格式的响应消息，用于AJAX请求返回。

#### 参数说明
- `$b`：操作结果标识（大于0表示成功，否则失败）
- `$msg`（可选）：自定义消息内容，默认为"成功！"

#### 返回值
无返回值，直接输出JSON并退出脚本。

#### 输出格式
成功时：
```json
{
    "result": 200,
    "message": "成功！",
    "id": 1
}
```

失败时：
```json
{
    "result": 500,
    "message": "失败！",  // 或自定义消息
    "id": 0
}
```

#### 使用示例
```php
// 成功操作
logmsg(1, "用户创建成功");
// 输出：{"result":200,"message":"用户创建成功","id":1}

// 失败操作
logmsg(0, "用户名已存在");
// 输出：{"result":500,"message":"用户名已存在","id":0}
```

#### 注意事项
- 函数执行后会调用 `exit`，后续代码不会执行
- 适用于AJAX接口返回

---

### 4. `get_token()` - Token生成和获取函数

#### 函数签名
```php
function get_token()
```

#### 功能描述
生成或获取CSRF防护Token，用于防止跨站请求伪造攻击。

#### 返回值
返回Token字符串（MD5哈希值）。

#### 工作流程
1. 检查Session中是否已存在Token
2. 如果不存在，使用 `md5(uniqid(rand(), true))` 生成新Token
3. 将Token存入Session：`$_SESSION[KEY.'token']`
4. 将Token存入Cookie：`md5(KEY.'token')` 作为Cookie名
5. Cookie有效期：24小时
6. 返回Token值

#### 代码问题
**注意**：代码中使用了未定义的常量 `KEY`，应该使用 `KEYS`：
```php
// 当前代码（有问题）
$_SESSION[KEY.'token']  // KEY未定义

// 应该改为
$_SESSION[KEYS.'token']  // 使用KEYS常量
```

#### 使用场景
- 用户登录时生成Token
- 发送消息前验证Token
- 防止CSRF攻击

---

### 5. `check_post($arr, $config = array())` - POST请求验证函数

#### 函数签名
```php
function check_post($arr, $config = array())
```

#### 功能描述
验证POST请求的Token，确保请求来自合法来源。

#### 参数说明
- `$arr`：请求数据数组（当前未使用）
- `$config`：配置数组（当前未使用）

#### 返回值
- `true`：Token验证通过
- `false`：Token验证失败

#### 工作流程
1. 从Cookie中获取Token：`$_COOKIE[md5(KEY.'token')]`
2. 从Session中获取存储的Token：`$_SESSION[KEY.'token']`
3. 比较两个Token是否一致
4. 如果Cookie中Token为空或与Session不一致，返回 `false`
5. 否则返回 `true`

#### 代码问题
**注意**：同样使用了未定义的 `KEY` 常量，应该改为 `KEYS`。

#### 使用示例
```php
$arr = array('msg' => '测试消息');
if (check_post($arr) == false) {
    // Token验证失败，拒绝请求
    exit('验证失败');
}
// Token验证通过，继续处理
```

#### 安全说明
- 这是CSRF防护机制
- Token存储在Session和Cookie中
- 每次请求都需要验证Token一致性

## 常量引用说明

### 在代码中使用的常量
- `KEYS`：Cookie和Session键前缀
- `ROOT_PATH`：项目根路径（用于文件操作）
- `MSGFILE`：消息文件路径

### 未定义但使用的常量（Bug）
- `KEY`：在 `get_token()` 和 `check_post()` 中使用，但未定义
  - **建议修复**：将所有 `KEY` 替换为 `KEYS`

## 使用方式

### 在其他文件中引入
```php
require_once 'app/app.php';
```

### 典型使用场景

#### 1. 在 index.php 中
```php
require_once 'app/app.php';
// 检查用户Cookie
if(empty(@$_COOKIE[KEYS.'_name'])){
    // 显示登录界面
}
```

#### 2. 在 ajax.php 中
```php
require_once 'app.php';
// 用户登录
$arr = nick($user);
// 生成Token
get_token();
// 验证请求
if(check_post($arr) == false){
    // 拒绝请求
}
```

## 已知问题和建议

### 1. 未定义变量问题
- **问题**：`rand_nick()` 函数中 `$name_tou` 和 `$name_wei` 未定义
- **影响**：随机昵称生成功能无法正常工作
- **建议**：定义完整的昵称前缀和后缀数组

### 2. 常量使用错误
- **问题**：`get_token()` 和 `check_post()` 中使用了未定义的 `KEY` 常量
- **影响**：Token验证功能可能无法正常工作
- **建议**：将 `KEY` 替换为 `KEYS`

### 3. 错误处理不足
- **问题**：文件操作没有错误处理
- **建议**：添加文件写入失败的错误处理机制

### 4. 安全性增强
- **建议**：对昵称和消息内容进行更严格的验证和过滤
- **建议**：考虑使用更安全的Token生成方式

## 依赖关系

### 被以下文件引用
- `index.php`：主入口页面
- `app/ajax.php`：AJAX请求处理

### 依赖的PHP功能
- `session_start()`：需要Session支持
- `file_put_contents()`：需要文件写入权限
- `setcookie()`：需要HTTP头未发送
- `date_default_timezone_set()`：需要时区设置权限

## 总结

`app.php` 是项目的核心基础库，提供了：
- 路径和配置常量
- 用户管理功能（昵称生成、Cookie设置）
- 安全验证功能（Token生成和验证）
- 工具函数（日志输出）

在使用时需要注意修复已知的Bug，并加强错误处理和安全验证。

