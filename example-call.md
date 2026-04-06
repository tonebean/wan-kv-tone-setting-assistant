# 品牌 KV 定调助手 — 最小调用示例

本文件提供基于 **Wan2.7 API** 的最小调用示例，用于说明本 Skill 最终如何落到实际接口调用。

当前示例以 **文生图主链路** 为主，采用异步任务流。

注意：  
这个文件只展示“最后怎么调接口”，不代表这套 skill 的真正特色。  
它真正有价值的部分，在前面的品牌任务判断、路线选择和定调过程里。  
如果首图已经对了但还差一点完成度，才进入局部校正；如果方向本身错了，优先回前面重选路线。

---

## 1. 调用目标

本 Skill 的完整链路为：

**用户 brief → 商业任务解析 → 视觉路线生成 → 执行描述 → Wan Prompt → 首图生成 → 校正判断**

本文件仅展示最后一段：

**Wan Prompt → Wan API 调用 → 图像结果**

如果要继续走局部校正，可再调用 Wan2.7 image 的图像编辑能力；但这一步只适合细节修正，不替代前面的品牌判断。

---

## 1.1 推荐回答口径

如果这套 skill 被真实用于对话，默认回答方式建议保持短、准、好录屏。

更适合的顺序是：

1. 先说任务判断
2. 再给 2 到 3 条路线
3. 明确推荐 1 条
4. 再给执行描述或出图结论
5. 最后只补一句下一步

不建议：

- 一上来长篇解释方法论
- 重复用户原话
- 路线给太多
- 每次都把完整流程重新讲一遍

更适合教程展示的表达像这样：

> 这次先做品牌定调，不先讲卖点。  
> 我会给你 3 条路线，推荐走更稳的这一条。  
> 如果你确认，我再直接出图。

---

## 2. 默认模型与接口

这份示例当前对应的是 **Wan 2.7 图像模型 API**。  
如果要核对上游能力边界，以这份官方文档入口为准：

- `https://waytoagi.feishu.cn/wiki/CNjhwJdNBiyiGpkuODMcn8R0nNd`

按上游文档，Wan2.7 image 还有组图、图像编辑、交互式编辑等能力。  
但这个示例只展示当前 skill 真正默认使用的一条链路：**PRO 版文生图**。

### 默认模型

- `wan2.7-image-pro`

### 默认接口

```text
POST https://dashscope.aliyuncs.com/api/v1/services/aigc/image-generation/generation
```

### 地域与鉴权提醒

- 本示例默认按北京地域写法展示。
- 北京与新加坡地域的 API Key 和请求地址不能混用。
- 推荐先按 `references/如何获取API.md` 获取 Key，再按 `references/API Key配置到环境变量.md` 配好 `DASHSCOPE_API_KEY`。
- 示例里统一从环境变量读取密钥，不建议把真实 Key 直接写进脚本。

### 请求头

```http
Content-Type: application/json
Authorization: Bearer $DASHSCOPE_API_KEY
X-DashScope-Async: enable
```

---

## 3. 最小 curl 调用示例

```bash
curl --location 'https://dashscope.aliyuncs.com/api/v1/services/aigc/image-generation/generation' \
  --header 'Content-Type: application/json' \
  --header "Authorization: Bearer $DASHSCOPE_API_KEY" \
  --header 'X-DashScope-Async: enable' \
  --data '{
    "model": "wan2.7-image-pro",
    "input": {
      "messages": [
        {
          "role": "user",
          "content": [
            {
              "text": "高端护肤精华广告主视觉，一支极简高级的精华瓶作为画面唯一主角，精致玻璃瓶身与金属细节，冷白色与银灰色主调，局部微弱香槟金高光，置于抽象未来感实验室空间中，画面洁净克制，大面积留白，稳定中轴构图，商业摄影风格，中近景 hero shot，冷白主光与轮廓边缘光精准勾勒瓶身与液体质地，透明亚克力结构与悬浮液体元素营造精密科技护肤氛围"
            }
          ]
        }
      ]
    },
    "parameters": {
      "size": "2K",
      "n": 1,
      "watermark": false,
      "thinking_mode": true
    }
  }'
```

补充说明：

- 官方接口在不显式传入时，`n` 可能按更高默认值执行；本 skill 的演示和验收默认固定 `n=1`，避免无意义多图花费。
- `size` 用 `2K` 适合大多数主视觉验证；如果要严格匹配横竖版，也可以直接写具体像素值。
- 若要做复现实验，可追加 `seed`；若要做多图参考、图像编辑或组图，不要继续沿用这份最小文生图模板。

---

## 4. 返回结果示意

接口返回后，通常会先拿到一个异步任务结果，其中最关键的是：

- `request_id`
- `task_id`
- `task_status`

示意结构如下：

```json
{
  "request_id": "xxxxxx",
  "output": {
    "task_id": "d492bffd-10b5-4169-b639-xxxxxx",
    "task_status": "RUNNING"
  }
}
```

此时需要继续查询任务状态。

---

## 5. 查询任务状态示例

```bash
curl -X GET "https://dashscope.aliyuncs.com/api/v1/tasks/$TASK_ID" \
  --header "Authorization: Bearer $DASHSCOPE_API_KEY"
```

任务完成后，返回结果中通常会包含：

- `task_status`
- `choices`
- `message.content[].image`

示意结构如下：

```json
{
  "request_id": "xxxxxx",
  "output": {
    "task_id": "d492bffd-10b5-4169-b639-xxxxxx",
    "task_status": "SUCCEEDED",
    "choices": [
      {
        "message": {
          "role": "assistant",
          "content": [
            {
              "image": "https://dashscope-result-xxx.aliyuncs.com/xxx.png",
              "type": "image"
            }
          ]
        }
      }
    ]
  },
  "usage": {
    "image_count": 1
  }
}
```

---

## 6. Python 最小调用示例

```python
import os
import requests

API_KEY = os.getenv("DASHSCOPE_API_KEY")
BASE_URL = "https://dashscope.aliyuncs.com/api/v1"

payload = {
    "model": "wan2.7-image-pro",
    "input": {
        "messages": [
            {
                "role": "user",
                "content": [
                    {
                        "text": "一张高质量商业海报，唯一主体，留白清晰，真实质感"
                    }
                ]
            }
        ]
    },
    "parameters": {
        "size": "2K",
        "n": 1,
        "watermark": False,
        "thinking_mode": True
    }
}

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {API_KEY}",
    "X-DashScope-Async": "enable",
}

resp = requests.post(
    f"{BASE_URL}/services/aigc/image-generation/generation",
    headers=headers,
    json=payload,
    timeout=60,
)
resp.raise_for_status()
result = resp.json()
print(result)
```

---

## 7. Python 查询任务示例

```python
import os
import requests

API_KEY = os.getenv("DASHSCOPE_API_KEY")
TASK_ID = "your-task-id"

headers = {
    "Authorization": f"Bearer {API_KEY}"
}

resp = requests.get(
    f"https://dashscope.aliyuncs.com/api/v1/tasks/{TASK_ID}",
    headers=headers,
    timeout=60,
)
resp.raise_for_status()
result = resp.json()
print(result)
```

---

## 8. 结果保存说明

Wan 图像生成结果通常具有时效性，因此建议：

1. 任务完成后立即下载结果图
2. 将图像保存到本地或对象存储
3. 演示用图不要依赖临时 URL 长时间存活

建议保存到：

- `./outputs/wanx/`

---

## 9. 与本 Skill 的关系

这些代码示例并不负责完整的商业视觉判断逻辑。  
本 Skill 的核心价值仍然在于：

- 需求梳理
- 路线生成
- 执行描述
- 最终 Prompt 构建

本文件展示的是该 Skill 最终如何落到 Wan2.7 的真实接口调用上。

---

## 10. 后续可扩展方向

在当前最小可用调用基础上，后续还可以继续扩展：

- 多图输入编辑
- 参考图风格统一
- seed 与可复现控制
- 自动结果下载与归档
- demo 批量生成脚本
