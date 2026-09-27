# 专栏投稿

::: warning 注意
旧版网页专栏编辑器 `https://member.bilibili.com/article/home` 已下线（返回 404），现役网页编辑器为 `https://member.bilibili.com/article-text/home`。本页端点于 2026-09 基于现役编辑器（Web 端）的前端代码分析与实测整理。
:::

专栏投稿流程：上传正文图片 → 上传封面 → 保存草稿 → 提交送审。保存草稿与提交送审共用同一组正文参数字段。

## 获取投稿预置信息

> https://member.bilibili.com/x/web/article/pre

*请求方式：GET*

认证方式：Cookie（SESSDATA）

专栏编辑器初始化接口，返回可选分类树与账号投稿状态。

**json回复：**

根对象：

| 字段    | 类型 | 内容     | 备注 |
| ------- | ---- | -------- | ---- |
| code    | num  | 返回值   | 0：成功 |
| message | str  | 错误信息 | 默认为 OK |
| ttl     | num  | 1        |      |
| data    | obj  | 信息本体 |      |

`data`对象：

| 字段       | 类型 | 内容             | 备注                                       |
| ---------- | ---- | ---------------- | ------------------------------------------ |
| categories | array | 可选分类树       | 结构同 [专栏分类](category.md)，含 `children` 子分类数组 |
| myinfo     | obj  | 账号投稿状态信息 |                                            |
| toplimit   | num  | 置顶数量上限？   | 作用尚不明确                               |

`myinfo`对象：

| 字段            | 类型 | 内容                     | 备注                                       |
| --------------- | ---- | ------------------------ | ------------------------------------------ |
| mid             | num  | 账号 mid                 |                                            |
| uname           | str  | 账号昵称                 |                                            |
| face            | str  | 账号头像 URL             |                                            |
| banned          | bool | 是否封禁中               |                                            |
| level           | num  | 等级                     |                                            |
| activated       | bool | 是否激活                 |                                            |
| deftime         | num  | 定时发布可用时间戳？     | 作用尚不明确                               |
| deftime_end     | num  | 定时发布可选上限时间戳？ | 作用尚不明确                               |
| deftime_msg     | str  | 定时发布限制提示         |                                            |
| commercial      | num  | 商业权限标记？           | 作用尚不明确                               |
| identify_check  | obj  | 实名认证状态             | 含 `code`、`msg`（如"已实名认证"）          |
| ai_type         | obj  | AI 创作声明相关标记      | 含 `white`、`black` 布尔位；作用尚不明确    |
| should_answer   | bool | 是否需要答题             |                                            |
| uploadsize      | num  | 上传大小限制             | 单位未确认                                 |
| arc_cnt         | num  | 稿件数                   |                                            |
| follower        | num  | 粉丝数                   |                                            |
| sign            | str  | 签名                     |                                            |

**示例：**

```shell
curl 'https://member.bilibili.com/x/web/article/pre' \
  -b 'SESSDATA=xxx'
```

<details>
<summary>查看响应示例（节选）：</summary>

```json
{
  "code": 0,
  "message": "OK",
  "ttl": 1,
  "data": {
    "categories": [
      { "id": 2, "parent_id": 0, "name": "动画", "children": [
        { "id": 4, "parent_id": 2, "name": "动漫杂谈" }
      ] }
    ],
    "myinfo": {
      "mid": 12345678,
      "uname": "******",
      "banned": false,
      "activated": true,
      "identify_check": { "code": 0, "msg": "已实名认证" },
      "ai_type": { "white": false, "black": false },
      "should_answer": false
    },
    "toplimit": 4
  }
}
```

</details>

## 上传专栏正文图片

> https://api.bilibili.com/x/dynamic/feed/draw/upload_bfs

*请求方式：POST*

认证方式：Cookie（SESSDATA）

与动态图片上传为同一端点（详见 [发布动态-为图片动态上传图片](../dynamic/publish.md)），专栏场景 `biz` 传 `article`。正文图片与自定义封面（二进制方式）均可使用该端点。

**正文参数（multipart/form-data）：**

| 参数名  | 类型   | 内容                       | 必要性 | 备注                                                        |
| ------- | ------ | -------------------------- | ------ | ----------------------------------------------------------- |
| file_up | file   | 需要上传的图片文件         | 必要   | 格式支持 `jpg` `png` `gif` `jpeg`；单图大小不超过 8MB        |
| biz     | string | 业务标识                   | 必要   | 专栏固定为 `article`                                        |
| csrf    | string | CSRF Token（位于 Cookie）  | 必要   | Cookie 中 `bili_jct`                                        |

**json回复：**

根对象：

| 字段    | 类型 | 内容     | 备注                            |
| ------- | ---- | -------- | ------------------------------- |
| code    | num  | 返回值   | 0：成功<br />-111：csrf 校验失败 |
| message | str  | 错误信息 | 成功时为 OK                     |
| ttl     | num  | 1        |                                 |
| data    | obj  | 信息本体 |                                 |

`data`对象：

| 字段         | 类型   | 内容            | 备注                            |
| ------------ | ------ | --------------- | ------------------------------- |
| image_url    | string | 已上传图片 URL  | `i0.hdslb.com/bfs/new_dyn/` 域  |
| image_width  | number | 图片宽度        | 像素                            |
| image_height | number | 图片高度        | 像素                            |
| img_size     | number | 图片大小        | KB                              |
| ai_gen_pic   | number | AI 生成图片标记 | 0：否；定义尚未明确（？）        |

**示例：**

```shell
curl 'https://api.bilibili.com/x/dynamic/feed/draw/upload_bfs' \
  -F 'file_up=@cover.png' \
  -F 'biz=article' \
  -F 'csrf=xxx' \
  -b 'SESSDATA=xxx'
```

<details>
<summary>查看响应示例：</summary>

```json
{
  "code": 0,
  "message": "OK",
  "ttl": 1,
  "data": {
    "image_url": "http://i0.hdslb.com/bfs/new_dyn/be3e6bd84aad5af577ba1deae6ef38c621587404.png",
    "image_width": 1920,
    "image_height": 1080,
    "img_size": 1281.463,
    "ai_gen_pic": 0
  }
}
```

</details>

## 上传专栏封面（base64）

> https://api.bilibili.com/x/article/creative/article/upcover

*请求方式：POST*

认证方式：Cookie（SESSDATA）

编辑器"自定义封面→上传封面"使用的 base64 变体端点；二进制上传可用上节 `upload_bfs`。

**正文参数（application/x-www-form-urlencoded）：**

| 参数名   | 类型   | 内容                      | 必要性 | 备注                                              |
| -------- | ------ | ------------------------- | ------ | ------------------------------------------------- |
| cover    | string | 封面图片 base64           | 必要   | 必须带 dataURL 前缀，如 `data:image/png;base64,`；裸 base64 会返回 -400 |
| card     | string | 分享卡片图 base64         | 不必要 | 编辑器会基于封面模板图生成分享卡片并一并上传       |
| filename | string | 文件名                    | 不必要 |                                                   |
| csrf     | string | CSRF Token（位于 Cookie） | 必要   | Cookie 中 `bili_jct`                              |

**json回复：**

根对象：

| 字段    | 类型 | 内容     | 备注                              |
| ------- | ---- | -------- | --------------------------------- |
| code    | num  | 返回值   | 0：成功<br />-400：请求错误        |
| message | str  | 错误信息 | 成功时为 OK                       |
| ttl     | num  | 1        |                                   |
| data    | obj  | 信息本体 |                                   |

`data`对象：

| 字段 | 类型   | 内容           | 备注                                     |
| ---- | ------ | -------------- | ---------------------------------------- |
| url  | string | 已上传封面 URL | `article.biliimg.com/bfs/new_dyn/` 域    |
| size | number | 图片大小       | 字节                                     |

## 保存专栏草稿

> https://api.bilibili.com/x/article/creative/draft/addupdate

*请求方式：POST*

认证方式：Cookie（SESSDATA）

新建或覆盖更新草稿。不传 `aid` 为新建草稿，传入已有草稿 `aid` 为覆盖更新。

**正文参数（application/x-www-form-urlencoded）：**

| 参数名            | 类型   | 内容                     | 必要性      | 备注                                                                 |
| ----------------- | ------ | ------------------------ | ----------- | -------------------------------------------------------------------- |
| title             | string | 专栏标题                 | 必要        |                                                                      |
| content           | string | 正文 HTML                | 必要        | 支持 `p` `img` `strong` 等标签子集                                   |
| banner_url        | string | 自定义封面 URL           | 非必要      | 封面图床 URL；与封面模板图（`image_urls`）二选一                      |
| summary           | string | 摘要                     | 非必要      |                                                                      |
| words             | number | 正文字数                 | 非必要      | 编辑器按正文纯文本统计                                                |
| category          | number | 分类 id                  | 必要        | 取值见 [专栏分类](category.md) 或预置信息接口                         |
| list_id           | number | 文集 id                  | 非必要      | `0` 为不选择文集                                                      |
| tid               | number | 封面模板 id              | 非必要      | `0` 为不使用封面模板                                                  |
| image_urls        | string | 封面模板裁剪图 URL       | 非必要      | 多张以逗号分隔，与 `origin_image_urls` 一一对应                        |
| origin_image_urls | string | 封面模板原图 URL         | 非必要      | 多张以逗号分隔                                                        |
| dynamic_intro     | string | 转发动态文案             | 非必要      |                                                                      |
| tags              | string | 话题标签                 | 非必要      | 多个以逗号分隔；空字符串为不添加话题                                   |
| reprint           | number | 是否转载                 | 非必要      | `0`：否<br />`1`：转载                                               |
| original          | number | 原创声明                 | 非必要      | `0`：不声明<br />`1`：声明原创；声明后正文图片可能附加防盗链保护       |
| top_video_bvid    | string | 置顶视频卡片 BV 号       | 非必要      | 空字符串为不插入；置顶视频卡片在正文前自动播放                         |
| media_id          | number | 媒体卡片 id              | 非必要      | 无则传 `0`                                                           |
| spoiler           | number | 是否含剧透               | 非必要      | `0`：否<br />`1`：是                                                 |
| private_pub       | number | 可见范围                 | 非必要      | `0`：所有人可见；其余取值待确认（？）                                 |
| up_reply_closed   | number | 关闭评论区               | 非必要      | `0`：开启评论<br />`1`：关闭评论                                     |
| comment_selected  | number | 评论精选                 | 非必要      | `0`：关闭<br />`1`：开启；开启评论时才可开精选                        |
| publish_time      | number | 定时发布时间戳           | 非必要      | 定时发布需晚于当前 2 小时且早于 15 天（？）                            |
| aid               | number | 草稿 id                  | 非必要      | 传入即覆盖更新该草稿                                                  |
| action            | number | 操作类型                 | 非必要      | `1`：仅保存（编辑器自动保存使用）？作用尚不明确                        |
| csrf              | string | CSRF Token（位于 Cookie）| 必要        | Cookie 中 `bili_jct`；缺失返回 -111                                  |

**json回复：**

根对象：

| 字段    | 类型 | 内容     | 备注                                                                 |
| ------- | ---- | -------- | -------------------------------------------------------------------- |
| code    | num  | 返回值   | 0：成功<br />-101：账号未登录<br />-111：csrf 校验失败<br />47030：草稿已失效（如已发布） |
| message | str  | 错误信息 | 成功时为 OK                                                          |
| ttl     | num  | 1        |                                                                      |
| data    | obj  | 草稿信息 | 含草稿 id（可用于 `draft/view` 读取）                                  |

## 读取草稿内容

> https://api.bilibili.com/x/article/creative/draft/view

*请求方式：GET*

认证方式：Cookie（SESSDATA）

**URL 参数：**

| 参数名 | 类型 | 内容    | 必要性 | 备注 |
| ------ | ---- | ------- | ------ | ---- |
| aid    | num  | 草稿 id | 必要   |      |

**json回复：**

根对象中 `data` 为草稿完整内容（字段与保存草稿参数基本对应，`reprint`、`original`、`category` 等以对象/原始值返回），`state` 为草稿状态（`0`：草稿箱中）。

## 草稿列表

> https://api.bilibili.com/x/article/creative/draft/list

*请求方式：GET*

认证方式：Cookie（SESSDATA）

**json回复：**

根对象：

| 字段    | 类型 | 内容     | 备注                                |
| ------- | ---- | -------- | ----------------------------------- |
| code    | num  | 返回值   | 0：成功                             |
| message | str  | 错误信息 | 默认为 OK                           |
| ttl     | num  | 1        |                                     |
| artlist | obj  | 草稿箱   | 无草稿时为 `null`                    |

`artlist`对象：

| 字段   | 类型  | 内容     | 备注                                   |
| ------ | ----- | -------- | -------------------------------------- |
| drafts | array | 草稿列表 | 每项含 `id`、`title`、`summary`、`state` 等 |

## 提交专栏送审

> https://api.bilibili.com/x/article/creative/article/submit

*请求方式：POST*

认证方式：Cookie（SESSDATA）

**URL 参数：**

| 参数名  | 类型   | 内容        | 必要性 | 备注 |
| ------- | ------ | ----------- | ------ | ---- |
| w_words | number | 正文字数    | 非必要 |      |
| w_buvid | string | 设备 buvid  | 非必要 |      |

**正文参数（application/x-www-form-urlencoded）：**

与 [保存专栏草稿](#保存专栏草稿) 相同，并需携带草稿 `aid`。

::: danger 注意
网页端提交时前端会附加 `UNIOS_RISK_USER_LOG` 与 `UNIOS_WBI_ENCODE` 客户端风控签名（未逆向），绕过编辑器直接调用本接口可能触发风控校验。建议用上述接口保存草稿后，在编辑器页面人工确认并点击发布。
:::

## 其他编辑器相关端点

以下端点见于现役编辑器前端代码，未逐一实测：

| 端点 | 用途推测 |
| ---- | -------- |
| `GET https://api.bilibili.com/x/article/creative/media/info?media_id=` | 查询媒体卡片信息（番剧/视频卡片） |
| `GET https://api.bilibili.com/x/article/creative/game/search` | 游戏卡片搜索 |
| `GET https://api.bilibili.com/x/article/creative/list/all` | 获取文集列表 |
| `GET https://api.bilibili.com/x/article/creative/list/add` | 新建文集 |
| `POST https://api.bilibili.com/x/article/creative/article/capture` | 正文外链抓图 |
| `POST https://api.bilibili.com/x/article/creative/article/upwatermark` | 上传水印图 |
| `POST https://api.bilibili.com/x/article/creative/article/update` | 更新已发布专栏（？） |

补充说明：

- 编辑器可插入的**正文内视频卡片**为编辑器交互生成，其在正文 HTML 中的序列化格式未逆向，建议通过编辑器插入。
- "AI 辅助创作声明"在现役网页编辑器前端中未发现对应提交字段（预置信息 `myinfo.ai_type` 为账号级标记），如需开启请在编辑器/App 端手动操作。
