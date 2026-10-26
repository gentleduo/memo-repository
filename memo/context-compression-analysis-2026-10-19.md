



# 背景

用户原始需求：“实现归并排序，编写并运行测试，最后编写 README。代码、测试、文档都放在 /workspace。”

| 原计划步骤 | 具体工作                                               | 终止后的状态                           |
| ---------- | ------------------------------------------------------ | -------------------------------------- |
| 1          | 编写 /workspace/merge_sort.py，实现归并排序            | COMPLETED                              |
| 2          | 编写并运行 /workspace/test_merge_sort.py，确认测试通过 | FAILED；测试脚本已创建，但尚未执行测试 |
| 3          | 编写 /workspace/README.md，说明用法和测试结果          | PENDING；未开始                        |

# 流程

## 先确定一个具体的错误场景

用户最初输入：

> 实现归并排序，编写并运行测试，最后编写 README。代码、测试、文档都放在 /workspace。

Planner 生成 `PLAN1`，系统建立任务 `T1`：

| 步骤   | 目标                                                     |
| ------ | -------------------------------------------------------- |
| Step 1 | 编写 `/workspace/merge_sort.py`，实现归并排序            |
| Step 2 | 编写并运行 `/workspace/test_merge_sort.py`，确认测试通过 |
| Step 3 | 编写 `/workspace/README.md`，说明用法和测试结果          |

实际过程是：

```markdown
Step 1
  写入 merge_sort.py 成功
  模型返回步骤成功结果
  保存步骤归档和检查点

Step 2
  写入 test_merge_sort.py 成功
  准备再次调用模型，继续安排测试
  模型服务调用失败
  重试后仍失败
  系统终结 Step 2 和整个任务

Step 3
  尚未开始
```

用户收到的错误是：

```markdown
{
  "code": "LLM_TRANSIENT_RETRY_EXHAUSTED",
  "error": "语言模型服务连续调用失败，当前任务已终止"
}
```

## 用户看到错误时，业务状态已经保存完成

此时几个“状态”分别是：

| 对象        | 状态        | 含义                                 |
| ----------- | ----------- | ------------------------------------ |
| Step 1      | `completed` | 第一步成功                           |
| Step 2      | `failed`    | 第二步失败                           |
| Step 3      | `pending`   | 第三步未执行                         |
| Plan        | `failed`    | 本轮计划因错误终止                   |
| Session     | `completed` | 本轮调度结束，不代表业务全部成功     |
| Task 检查点 | `PARTIAL`   | 三步中已经完成一步，整个任务部分完成 |

两个文件也保留下来：

| 文件                            | 已确认的事实                     |
| ------------------------------- | -------------------------------- |
| `/workspace/merge_sort.py`      | 已创建，336 字节                 |
| `/workspace/test_merge_sort.py` | 已创建，171 字节，但尚未运行测试 |
| `/workspace/README.md`          | 尚未创建                         |

### agent_step_archives

保存已执行结束的步骤的交互消息原文。但是不能说是所有大模型交互的完整记录，因为它不包含：

1. 固定 ReAct system 提示词，因为 system 位于 Step 工作消息范围之前。
2. 每次请求临时拼接的项目连续性、步骤进度投影。
3. 临时的协议纠正或重复调用提醒。
4. 请求里的工具 schema、`response_format` 等其他参数。
5. 每一次网络重试的独立请求记录。

Planner 自己的完整交互流水。

```json
{
    "agent_step_archives": [
    {
      "archive_id": "c80f1210-d505-49c0-88cb-ffd55ce713d0",
      "project_id": "project-1",
      "payload": {
        "archive_id": "c80f1210-d505-49c0-88cb-ffd55ce713d0",
        "project_id": "project-1",
        "task_execution_id": "41cad2e8-c024-45dc-875b-85be0e282af8",
        "plan_id": "2d5e1df4-26f4-4e14-a8c1-a36568dd4272",
        "step_id": "1",
        "attempt_no": 1,
        "epoch": 1,
        "start_sequence": 2,
        "end_sequence": 5,
        "messages": [
          {
            "role": "user",
            "content": "\n你正在执行任务：\n编写 /workspace/merge_sort.py，实现归并排序\n\n注意事项：\n- **是你来执行这个任务，而不是用户。**不要告诉用户“如何做”，而是直接通过工具“去做”。\n- **必须使用用户消息中使用的语言（Working Language）来执行任务和回复。**\n- 必须使用 `message_notify_user` 工具向用户通报进度，内容限制在一句话以内：\n    - 你打算使用什么工具，以及用它做什么；\n    - 或者你通过工具完成了什么；\n    - 简明扼要地告知当前动作。\n- 如果你需要用户提供输入或需要获取浏览器的控制权，必须使用 `message_ask_user` 工具向用户提问。\n- 再次强调：直接交付最终结果，而不是提供待办事项列表、建议或计划。\n\n\n返回格式要求：\n- 当你不再需要调用工具并准备结束当前步骤时，必须只返回一个 JSON 对象，不要返回 Markdown、自然语言段落、代码块或 JSON 之外的解释。\n- JSON 对象必须包含 `success`、`work_summary`、`result`、`attachments` 四个字段；当前步骤失败时也必须返回 JSON，并在 `result` 中说明失败原因。\n- `work_summary` 用于告诉后续步骤本步骤具体完成了什么，必须保持完整语义且简洁：说明实际完成的工作、关键验证结论，以及失败时仍未解决的问题；不要重复任务目标、工具调用流水或大段功能清单，建议控制在 100～300 个汉字以内。\n- 即使当前步骤只是读取技能定义或确认执行规范，也必须按 JSON 格式返回，不要用普通自然语言收尾。\n- 必须返回符合以下 TypeScript 接口定义的 JSON 格式。\n- 必须包含所有指定的必填字段。\n\nTypeScript 接口定义：\n```typescript\ninterface Response {\n  /** 任务步骤是否成功执行 **/\n  success: boolean;\n  /** 本步骤实际完成工作的短摘要，供后续步骤继续执行时使用 **/\n  work_summary: string;\n  /** 沙箱中需要交付给用户的生成文件的路径数组 **/\n  attachments: string[];\n\n  /** 任务结果文本，如果没有结果需要交付则留空 **/\n  result: string;\n}\n```\n\nJSON 输出示例：\n{\n    \"success\": true,\n    \"work_summary\": \"已清洗客户数据并生成统计摘要，空值和重复记录处理结果已核对。\",\n    \"result\": \"我们已经完成了数据清洗任务，并生成了摘要。\",\n    \"attachments\": [\n        \"/workspace/file1.md\",\n        \"/workspace/file2.md\"\n    ]\n}\n\n仅完成技能定义读取时的正确输出示例：\n{\n    \"success\": true,\n    \"work_summary\": \"已读取 Python 开发技能定义，后续步骤可按照其中的工程规范重新规划和执行。\",\n    \"result\": \"已读取技能定义，并将按照该技能继续执行后续任务。\",\n    \"attachments\": []\n}\n\n\n输入信息：\n- message: 用户消息（请在所有文本输出中使用此语言）\n- attachments: 用户提供的附件\n- language: 当前的工作语言\n- task: 当前需要执行的任务\n\n输出：\n- JSON 格式的步骤执行结果\n\n用户消息(message):\n实现归并排序，编写并运行测试，最后编写 README。代码、测试、文档都放在 /workspace。\n\n附件(attachments):\n\n\n工作语言(language):\nzh\n\n任务(task):\n编写 /workspace/merge_sort.py，实现归并排序\n",
            "_context_sequence": 2,
            "_context_epoch": 1
          },
          {
            "role": "assistant",
            "content": "",
            "tool_calls": [
              {
                "id": "call-write-merge",
                "type": "function",
                "function": {
                  "name": "write_file",
                  "arguments": "{\"filepath\":\"/workspace/merge_sort.py\",\"content\":\"def merge_sort(values):\\n    if len(values) < 2:\\n        return list(values)\\n    middle = len(values) // 2\\n    left = merge_sort(values[:middle])\\n    right = merge_sort(values[middle:])\\n    result = []\\n    while left and right:\\n        result.append(left.pop(0) if left[0] <= right[0] else right.pop(0))\\n    return result + left + right\\n\",\"purpose\":\"归并排序实现\"}"
                }
              }
            ],
            "_context_sequence": 3,
            "_context_epoch": 1
          },
          {
            "role": "tool",
            "tool_call_id": "call-write-merge",
            "function_name": "write_file",
            "content": "{\"success\":true,\"message\":\"写入成功\",\"data\":{\"created\":true,\"filepath\":\"/workspace/merge_sort.py\",\"utf8_bytes\":336}}",
            "_context_tool_executed": true,
            "_context_artifact_facts": [
              {
                "path": "/workspace/merge_sort.py",
                "state": "CREATED",
                "media_type": "text/x-python",
                "utf8_bytes": 336,
                "purpose": "归并排序实现",
                "source_sequence": 4
              }
            ],
            "_context_sequence": 4,
            "_context_epoch": 1
          },
          {
            "role": "assistant",
            "content": "{\"id\": \"1\", \"description\": \"编写 /workspace/merge_sort.py，实现归并排序\", \"success\": true, \"result\": \"已创建归并排序实现。\", \"work_summary\": \"已创建 /workspace/merge_sort.py，包含归并排序实现。\", \"attachments\": []}",
            "_context_sequence": 5,
            "_context_epoch": 1
          }
        ],
        "sha256": "350eb721d3627ec0da84cdc27d15f63dc18d9901e28ebde6b25ab07318ba575c",
        "utf8_bytes": 4968,
        "reason": "COMPLETED_STEP",
        "created_at": "2026-10-21T17:07:46.004760+08:00"
      },
      "created_at": "2026-10-21T17:07:46.004760+08:00"
    },
    {
      "archive_id": "7e3d69a2-1936-4fe0-85c9-76be66af3c88",
      "project_id": "project-1",
      "payload": {
        "archive_id": "7e3d69a2-1936-4fe0-85c9-76be66af3c88",
        "project_id": "project-1",
        "task_execution_id": "41cad2e8-c024-45dc-875b-85be0e282af8",
        "plan_id": "2d5e1df4-26f4-4e14-a8c1-a36568dd4272",
        "step_id": "2",
        "attempt_no": 1,
        "epoch": 1,
        "start_sequence": 6,
        "end_sequence": 8,
        "messages": [
          {
            "role": "user",
            "content": "\n你正在执行任务：\n编写并运行 /workspace/test_merge_sort.py，确认测试通过\n\n注意事项：\n- **是你来执行这个任务，而不是用户。**不要告诉用户“如何做”，而是直接通过工具“去做”。\n- **必须使用用户消息中使用的语言（Working Language）来执行任务和回复。**\n- 必须使用 `message_notify_user` 工具向用户通报进度，内容限制在一句话以内：\n    - 你打算使用什么工具，以及用它做什么；\n    - 或者你通过工具完成了什么；\n    - 简明扼要地告知当前动作。\n- 如果你需要用户提供输入或需要获取浏览器的控制权，必须使用 `message_ask_user` 工具向用户提问。\n- 再次强调：直接交付最终结果，而不是提供待办事项列表、建议或计划。\n\n\n返回格式要求：\n- 当你不再需要调用工具并准备结束当前步骤时，必须只返回一个 JSON 对象，不要返回 Markdown、自然语言段落、代码块或 JSON 之外的解释。\n- JSON 对象必须包含 `success`、`work_summary`、`result`、`attachments` 四个字段；当前步骤失败时也必须返回 JSON，并在 `result` 中说明失败原因。\n- `work_summary` 用于告诉后续步骤本步骤具体完成了什么，必须保持完整语义且简洁：说明实际完成的工作、关键验证结论，以及失败时仍未解决的问题；不要重复任务目标、工具调用流水或大段功能清单，建议控制在 100～300 个汉字以内。\n- 即使当前步骤只是读取技能定义或确认执行规范，也必须按 JSON 格式返回，不要用普通自然语言收尾。\n- 必须返回符合以下 TypeScript 接口定义的 JSON 格式。\n- 必须包含所有指定的必填字段。\n\nTypeScript 接口定义：\n```typescript\ninterface Response {\n  /** 任务步骤是否成功执行 **/\n  success: boolean;\n  /** 本步骤实际完成工作的短摘要，供后续步骤继续执行时使用 **/\n  work_summary: string;\n  /** 沙箱中需要交付给用户的生成文件的路径数组 **/\n  attachments: string[];\n\n  /** 任务结果文本，如果没有结果需要交付则留空 **/\n  result: string;\n}\n```\n\nJSON 输出示例：\n{\n    \"success\": true,\n    \"work_summary\": \"已清洗客户数据并生成统计摘要，空值和重复记录处理结果已核对。\",\n    \"result\": \"我们已经完成了数据清洗任务，并生成了摘要。\",\n    \"attachments\": [\n        \"/workspace/file1.md\",\n        \"/workspace/file2.md\"\n    ]\n}\n\n仅完成技能定义读取时的正确输出示例：\n{\n    \"success\": true,\n    \"work_summary\": \"已读取 Python 开发技能定义，后续步骤可按照其中的工程规范重新规划和执行。\",\n    \"result\": \"已读取技能定义，并将按照该技能继续执行后续任务。\",\n    \"attachments\": []\n}\n\n\n输入信息：\n- message: 用户消息（请在所有文本输出中使用此语言）\n- attachments: 用户提供的附件\n- language: 当前的工作语言\n- task: 当前需要执行的任务\n\n输出：\n- JSON 格式的步骤执行结果\n\n用户消息(message):\n实现归并排序，编写并运行测试，最后编写 README。代码、测试、文档都放在 /workspace。\n\n附件(attachments):\n\n\n工作语言(language):\nzh\n\n任务(task):\n编写并运行 /workspace/test_merge_sort.py，确认测试通过\n",
            "_context_sequence": 6,
            "_context_epoch": 1
          },
          {
            "role": "assistant",
            "content": "",
            "tool_calls": [
              {
                "id": "call-write-test",
                "type": "function",
                "function": {
                  "name": "write_file",
                  "arguments": "{\"filepath\":\"/workspace/test_merge_sort.py\",\"content\":\"from merge_sort import merge_sort\\n\\nassert merge_sort([]) == []\\nassert merge_sort([3, 1, 2]) == [1, 2, 3]\\nassert merge_sort([2, 1, 2]) == [1, 2, 2]\\nprint(\\\"3 tests passed\\\")\\n\",\"purpose\":\"归并排序测试脚本\"}"
                }
              }
            ],
            "_context_sequence": 7,
            "_context_epoch": 1
          },
          {
            "role": "tool",
            "tool_call_id": "call-write-test",
            "function_name": "write_file",
            "content": "{\"success\":true,\"message\":\"写入成功\",\"data\":{\"created\":true,\"filepath\":\"/workspace/test_merge_sort.py\",\"utf8_bytes\":171}}",
            "_context_tool_executed": true,
            "_context_artifact_facts": [
              {
                "path": "/workspace/test_merge_sort.py",
                "state": "CREATED",
                "media_type": "text/x-python",
                "utf8_bytes": 171,
                "purpose": "归并排序测试脚本",
                "source_sequence": 8
              }
            ],
            "_context_sequence": 8,
            "_context_epoch": 1
          }
        ],
        "sha256": "8cdd48f86ef7a480eaa980c0adf512a40db8c55cf408453828f51addaac685b3",
        "utf8_bytes": 4513,
        "reason": "COMPLETED_STEP",
        "created_at": "2026-10-21T17:07:46.012308+08:00"
      },
      "created_at": "2026-10-21T17:07:46.012308+08:00"
    }
  ]
}
```

>- `agent_step_archives` 此时有两份原文归档。
>
>  | 归档 | 任务/步骤   | attempt | epoch | 消息范围 | reason           |
>  | ---- | ----------- | ------- | ----- | -------- | ---------------- |
>  | A1   | T1 / Step 1 | 1       | 1     | 2～5     | `COMPLETED_STEP` |
>  | A2   | T1 / Step 2 | 1       | 1     | 6～8     | `COMPLETED_STEP` |
>
>- attempt_no：表示同一step第几次执行，比如：
>
>  ```markdown
>  整个任务尚未终止、Planner在任务内部安排重试：
>  Task：T1
>  Plan：P1
>  Step：2，运行测试并修复问题
>  
>  第一次执行：
>  执行 Step 2
>  → 测试发现错误
>  → 模型返回 success=false
>  → Step 2 本次执行终结为 FAILED
>  → 保存attempt_no=1的失败检查点
>  
>  然后Planner更新计划，决定：
>  仍然使用 step_id=2
>  将步骤内容调整为“修复已发现的问题并重新测试”
>  重新执行这个步骤
>  
>  第二次建立执行边界时：
>  task_execution_id 仍然是 T1
>  step_id 仍然是 2
>  attempt_no 变为 2
>  ```

### agent_context_checkpoints

步骤、任务和项目的结构化状态及结果；不保存完整的消息流水。

为方便理解，简称为 `CP-S1`、`CP-S2`、`CP-T1`、`CP-P1`：

| 记录  | checkpoint_type      | scope_id  | parent_scope_id | revision | status      |
| ----- | -------------------- | --------- | --------------- | -------- | ----------- |
| CP-S1 | `STEP`               | `"1"`     | T1              | 1        | `COMPLETED` |
| CP-S2 | `STEP`               | `"2"`     | T1              | 1        | `FAILED`    |
| CP-T1 | `TASK_COMPLETION`    | T1        | project-1       | 1        | `PARTIAL`   |
| CP-P1 | `PROJECT_CONTINUITY` | project-1 | null            | 1        | `CURRENT`   |

```json
{
  "agent_context_checkpoints": [
    {
      "project_id": "project-1",
      "checkpoint_type": "STEP",
      "scope_id": "1",
      "parent_scope_id": "41cad2e8-c024-45dc-875b-85be0e282af8",
      "revision": 1,
      "status": "COMPLETED",
      "payload": {
        "checkpoint_id": "bda48ae9-8d58-4c0a-a126-69cbbf1577bf",
        "project_id": "project-1",
        "task_execution_id": "41cad2e8-c024-45dc-875b-85be0e282af8",
        "plan_id": "2d5e1df4-26f4-4e14-a8c1-a36568dd4272",
        "step_id": "1",
        "attempt_no": 1,
        "revision": 1,
        "status": "COMPLETED",
        "objective": "编写 /workspace/merge_sort.py，实现归并排序",
        "success": true,
        "result_summary": "已创建 /workspace/merge_sort.py，包含归并排序实现。",
        "artifacts": [
          {
            "path": "/workspace/merge_sort.py",
            "state": "CREATED",
            "media_type": "text/x-python",
            "utf8_bytes": 336,
            "purpose": "归并排序实现"
          }
        ],
        "failure": null,
        "source_archive_ids": [
          "c80f1210-d505-49c0-88cb-ffd55ce713d0"
        ],
        "created_at": "2026-10-21T17:07:46.004855+08:00",
        "updated_at": "2026-10-21T17:07:46.004867+08:00"
      },
      "created_at": "2026-10-21 17:07:46.004855+08:00",
      "updated_at": "2026-10-21 17:07:46.004867+08:00",
      "id": "bda48ae9-8d58-4c0a-a126-69cbbf1577bf"
    },
    {
      "project_id": "project-1",
      "checkpoint_type": "STEP",
      "scope_id": "2",
      "parent_scope_id": "41cad2e8-c024-45dc-875b-85be0e282af8",
      "revision": 1,
      "status": "FAILED",
      "payload": {
        "checkpoint_id": "a30392f5-9472-4d3a-90e3-514e11769466",
        "project_id": "project-1",
        "task_execution_id": "41cad2e8-c024-45dc-875b-85be0e282af8",
        "plan_id": "2d5e1df4-26f4-4e14-a8c1-a36568dd4272",
        "step_id": "2",
        "attempt_no": 1,
        "revision": 1,
        "status": "FAILED",
        "objective": "编写并运行 /workspace/test_merge_sort.py，确认测试通过",
        "success": false,
        "result_summary": "语言模型服务连续调用失败，当前任务已终止",
        "artifacts": [
          {
            "path": "/workspace/test_merge_sort.py",
            "state": "CREATED",
            "media_type": "text/x-python",
            "utf8_bytes": 171,
            "purpose": "归并排序测试脚本"
          }
        ],
        "failure": {
          "code": "LLM_TRANSIENT_RETRY_EXHAUSTED",
          "summary": "语言模型服务连续调用失败，当前任务已终止"
        },
        "source_archive_ids": [
          "7e3d69a2-1936-4fe0-85c9-76be66af3c88"
        ],
        "created_at": "2026-10-21T17:07:46.012412+08:00",
        "updated_at": "2026-10-21T17:07:46.012423+08:00"
      },
      "created_at": "2026-10-21 17:07:46.012412+08:00",
      "updated_at": "2026-10-21 17:07:46.012423+08:00",
      "id": "a30392f5-9472-4d3a-90e3-514e11769466"
    },
    {
      "project_id": "project-1",
      "checkpoint_type": "TASK_COMPLETION",
      "scope_id": "41cad2e8-c024-45dc-875b-85be0e282af8",
      "parent_scope_id": "project-1",
      "revision": 1,
      "status": "PARTIAL",
      "payload": {
        "checkpoint_id": "6e87d8a5-0c96-4a70-9332-cb7525d9c3a6",
        "project_id": "project-1",
        "task_execution_id": "41cad2e8-c024-45dc-875b-85be0e282af8",
        "plan_id": "2d5e1df4-26f4-4e14-a8c1-a36568dd4272",
        "status": "PARTIAL",
        "goal": "实现归并排序，完成测试并提供 README",
        "success": false,
        "result_summary": "已创建 /workspace/merge_sort.py，包含归并排序实现。；语言模型服务连续调用失败，当前任务已终止",
        "outcome": {
          "summary": "任务部分完成（1/3步）。已完成：已创建 /workspace/merge_sort.py，包含归并排序实现。未完成：编写并运行 /workspace/test_merge_sort.py，确认测试通过；编写 /workspace/README.md，说明用法和测试结果。终止原因：语言模型服务连续调用失败，当前任务已终止。",
          "artifacts": [
            {
              "path": "/workspace/merge_sort.py",
              "purpose": "归并排序实现",
              "state": "CREATED"
            },
            {
              "path": "/workspace/test_merge_sort.py",
              "purpose": "归并排序测试脚本",
              "state": "CREATED"
            }
          ]
        },
        "artifacts": [
          {
            "path": "/workspace/merge_sort.py",
            "state": "CREATED",
            "media_type": "text/x-python",
            "utf8_bytes": 336,
            "purpose": "归并排序实现"
          },
          {
            "path": "/workspace/test_merge_sort.py",
            "state": "CREATED",
            "media_type": "text/x-python",
            "utf8_bytes": 171,
            "purpose": "归并排序测试脚本"
          }
        ],
        "step_checkpoint_ids": [
          "bda48ae9-8d58-4c0a-a126-69cbbf1577bf",
          "a30392f5-9472-4d3a-90e3-514e11769466"
        ],
        "task_context": {
          "inherited_from_task_execution_id": null,
          "goal": "实现归并排序，完成测试并提供 README",
          "user_requests": [
            "实现归并排序，编写并运行测试，最后编写 README。代码、测试、文档都放在 /workspace。"
          ],
          "existing_work": "",
          "related_files": [
            {
              "path": "/workspace/merge_sort.py",
              "purpose": "归并排序实现"
            },
            {
              "path": "/workspace/test_merge_sort.py",
              "purpose": "归并排序测试脚本"
            }
          ]
        },
        "created_at": "2026-10-21T17:07:46.013252+08:00"
      },
      "created_at": "2026-10-21 17:07:46.013252+08:00",
      "updated_at": "2026-10-21 17:07:46.013252+08:00",
      "id": "6e87d8a5-0c96-4a70-9332-cb7525d9c3a6"
    },
    {
      "project_id": "project-1",
      "checkpoint_type": "PROJECT_CONTINUITY",
      "scope_id": "project-1",
      "parent_scope_id": null,
      "revision": 1,
      "status": "CURRENT",
      "payload": {
        "checkpoint_id": "33870f47-270f-4e5a-95c1-5632d6f0c483",
        "project_id": "project-1",
        "revision": 1,
        "artifacts": [
          {
            "path": "/workspace/merge_sort.py",
            "state": "CREATED",
            "media_type": "text/x-python",
            "utf8_bytes": 336,
            "purpose": "归并排序实现"
          },
          {
            "path": "/workspace/test_merge_sort.py",
            "state": "CREATED",
            "media_type": "text/x-python",
            "utf8_bytes": 171,
            "purpose": "归并排序测试脚本"
          }
        ],
        "recent_tasks": [
          {
            "task_execution_id": "41cad2e8-c024-45dc-875b-85be0e282af8",
            "goal": "实现归并排序，完成测试并提供 README",
            "status": "PARTIAL",
            "success": false,
            "result_summary": "已创建 /workspace/merge_sort.py，包含归并排序实现。；语言模型服务连续调用失败，当前任务已终止",
            "outcome": {
              "summary": "任务部分完成（1/3步）。已完成：已创建 /workspace/merge_sort.py，包含归并排序实现。未完成：编写并运行 /workspace/test_merge_sort.py，确认测试通过；编写 /workspace/README.md，说明用法和测试结果。终止原因：语言模型服务连续调用失败，当前任务已终止。",
              "artifacts": [
                {
                  "path": "/workspace/merge_sort.py",
                  "purpose": "归并排序实现",
                  "state": "CREATED"
                },
                {
                  "path": "/workspace/test_merge_sort.py",
                  "purpose": "归并排序测试脚本",
                  "state": "CREATED"
                }
              ]
            }
          }
        ],
        "last_applied_task_checkpoint_id": "6e87d8a5-0c96-4a70-9332-cb7525d9c3a6",
        "updated_at": "2026-10-21T17:07:46.013395+08:00"
      },
      "created_at": "2026-10-21 17:07:46.013395+08:00",
      "updated_at": "2026-10-21 17:07:46.013395+08:00",
      "id": "33870f47-270f-4e5a-95c1-5632d6f0c483"
    }
  ]
}
```

>- TASK_COMPLETION和PROJECT_CONTINUITY中的outcome.summary 由代码生成
>
>- STEP中的source_archive_ids内存放归档ID，指向：agent_step_archives.archive_id，也就是产生该检查点结果的依据
>
>- epoch：这次执行属于哪一个版本的上下文。
>
>- attempt和epoch的区别
>
>  1. 换上下文时，业务尝试没有结束，所以attempt不增加；
>  2. 重新执行一个已终结的失败步骤时，attempt 增加，但epoch不必变化；
>  3. STEP的检查点通常不需要单独记录epoch，因为一个Attempt可能多个epoch的上下文
>  4. agent_context_checkpoints 是多类型表：STEP 类型有 attempt_no；TASK_COMPLETION、PROJECT_CONTINUITY 没有这个字段。
>
>  | 时刻                         | attempt    | epoch | 新增的归档/检查点                                |
>  | ---------------------------- | ---------- | ----- | ------------------------------------------------ |
>  | T1 的 Step 2 首次执行        | 1          | 1     | 暂无                                             |
>  | 本次执行失败                 | 1          | 1     | A1：attempt 1、epoch 1；C1：attempt 1、FAILED    |
>  | Planner 安排相同 Step 重试   | 2          | 1     | 建立新 active，尚不新增归档                      |
>  | 重试中上下文超限，切换上下文 | **仍为 2** | 1→2   | A2：归档 attempt 2、epoch 1 的原文               |
>  | 在新上下文中继续并成功       | 2          | 2     | A3：attempt 2、epoch 2；C2：attempt 2、COMPLETED |
>
>- `CP-T1.task_context` 保存完整需求：后面“请继续”能够继承原始需求，主要依靠这里保存的 task_context。
>
>- PROJECT_CONTINUITY：项目连续检查点，主要保存：
>
>  ```markdown
>  revision：1
>  artifacts：两个已创建文件
>  recent_tasks：包含 T1 的目标、PARTIAL 状态、结果摘要和 outcome
>  last_applied_task_checkpoint_id：CP-T1
>  ```
>
>  它用于后续任务快速了解这个项目已经有哪些成果、最近发生过什么。它本身不包含完整的原始需求链。

### agent_runtime_states

保存当前执行步骤的工作状态，这部分不仅保存“已经完成的交互”，还保存“正在执行、尚未完成的动作”。主要目的是保证进程异常退出后，新的进程通过加载数据库知道当前步骤中，那些工具已经完成，结果是什么；哪些工具异常结束，哪些工具尚未执行。

```json
{
  "agent_runtime_states": [
    {
      "project_id": "project-1",
      "state_type": "PLANNER_MEMORY",
      "revision": 5,
      "payload": {
        "schema_version": 7,
        "revision": 5,
        "context_epoch": 1,
        "next_sequence": 6,
        "messages": [
          {
            "role": "system",
            "content": "\n你是 Dev Mate，一个面向企业内网工程开发场景的专业代码助手。\n\n<intro>\n你的专长在于处理以下任务：\n- 信息收集、事实核查和文档撰写\n- 数据处理、分析和可视化\n- 撰写多章节长篇文章和深度研究报告\n- 利用编程解决软件开发以外的各类问题\n- 各种可以通过计算机、公司内网、本地文件和已配置工具完成的任务\n</intro>\n\n<language_settings>\n- 默认工作语言：**中文 (Chinese)**\n- 当用户在消息中明确指定语言时，使用用户指定的语言作为工作语言\n- 所有的思考过程（Thinking）和回复必须使用工作语言\n- 工具调用（Tool calls）中的自然语言参数必须使用工作语言\n- 在任何语言中，都要避免使用纯列表（List）和要点（Bullet points）格式\n</language_settings>\n\n<system_capability>\n- 能够访问不具有互联网连接的 Linux 沙箱环境\n- 可以使用 Shell、文本编辑器、浏览器和其他软件\n- 能够编写并运行 Python 及各种编程语言的代码\n- 仅能使用Linux 沙箱环境已有的软件，无法通过shell安装额外的软件包和依赖项\n- 必要时，建议用户在进行敏感操作时暂时接管浏览器控制权\n- 利用各种工具分步骤完成用户分配的任务\n</system_capability>\n\n<file_rules>\n- **必须**使用文件工具进行读取、写入、追加和编辑，以避免 Shell 命令中出现的字符串转义问题\n- 所有用户可见的代码、文档、数据、图表和中间产物都必须保存到 `/workspace` 或其子目录\n- `/home/athena` 是旧通用智能体路径，当前代码助手禁止把交付文件写入 `/home/athena`\n- 主动保存中间结果，并将不同类型的参考信息存储在单独的文件中\n- 合并文本文件时，必须使用文件写入工具的“追加模式”将内容连接到目标文件\n- 严格遵守 <writing_rules> 中的要求，除 `todo.md` 外，避免在任何文件中使用列表格式\n- 不要读取非文本文件、非代码文件或非 Markdown 文件\n</file_rules>\n\n<search_rules>\n- 由于无法访问互联网，所以请不要使用专用的搜索工具\n- 用户提供了可访问的内网 URL，或任务明确允许访问的资料源时，请使用浏览器访问\n- 信息优先级：**用户提供的文件/数据、公司内网系统、已配置内部知识源 > 可访问网络来源 > 模型的内部知识**\n- 搜索结果中的“摘要（Snippets）”不是有效来源；如果可以访问原始页面，应通过浏览器访问原始页面进行核验\n- 如果无法访问外部资料，应明确说明受内网环境限制，并请用户提供必要文件、数据或可访问的内部链接\n</search_rules>\n\n<browser_rules>\n- 必须使用浏览器工具访问并理解用户在消息中提供的所有 URL\n- 必须使用浏览器工具访问搜索工具结果中的 URL\n- 主动探索有价值的链接以获取更深层信息（通过点击元素或直接访问 URL）\n- 浏览器工具默认仅返回可见视口（Viewport）中的元素\n- 可见元素返回格式为 `index[:]<tag>text</tag>`，其中 `index` 用于后续浏览器操作中的交互\n- 由于技术限制，可能无法识别所有交互元素；对于未列出的元素，请使用坐标进行交互\n- 浏览器工具会自动尝试提取页面内容，如果成功则提供 Markdown 格式\n- 提取的 Markdown 包含视口之外的文本，但会省略链接和图像；不保证内容的完整性\n- 如果提取的 Markdown 完整且足以完成任务，则无需滚动；否则，必须主动滚动页面以查看完整内容\n</browser_rules>\n\n<shell_rules>\n- 避免使用需要用户确认的命令；主动使用 `-y` 或 `-f` 标志进行自动确认\n- 避免产生过多输出的命令；必要时将输出保存到文件中\n- 使用 `&&` 运算符链接多个命令，以尽量减少中断\n- 使用管道运算符（Pipe operator）传递命令输出，简化操作流程\n- 简单计算使用非交互式的 `bc` 命令，复杂数学计算编写 Python 代码；**切勿进行心算**\n- 当用户明确请求检查沙箱状态或唤醒时，使用 `uptime` 命令\n- 执行命令的工作目录必须是 `/workspace`；命令中不要写入 `/home/athena`\n</shell_rules>\n\n<coding_rules>\n- 代码执行前**必须**保存到文件中；禁止直接向解释器命令输入代码\n- 代码文件、测试文件和运行结果必须保存在 `/workspace` 下，便于右侧 Workspace 下载和预览\n- 编写 Python 代码进行复杂的数学计算和数据分析\n- 遇到不熟悉的问题时，由于无法访问互联网请勿使用搜索工具寻找解决方案\n</coding_rules>\n\n<writing_rules>\n- 使用连续的段落编写内容，采用长短句结合的方式使行文流畅生动；**严禁使用列表格式**\n- 默认使用散文和段落形式；仅在用户明确要求时才使用列表\n- 基于参考资料写作时，主动引用带有来源的原文，并在文末提供包含 URL 的参考文献列表\n- 对于长篇文档，先将每个部分保存为单独的草稿文件，然后按顺序追加合并为最终文档\n- 在最终汇编过程中，**不得删减或总结内容**；最终文档的长度必须超过所有单个草稿文件的总和\n</writing_rules>\n\n<sandbox_environment>\n系统环境:\n- Ubuntu 22.04 (linux/amd64)，公司办公环境中默认不具备公网互联网访问权限，仅能访问公司内网、本地服务和预置依赖\n- 用户: `athena`，拥有 sudo 权限\n- 工程工作目录: /workspace\n- 交付文件目录: /workspace\n\n开发环境:\n- Python 3.14 (命令: python3, python, pip3)\n- Node.js 20.18.0 (命令: node, npm)\n- 基础计算器 (命令: bc)\n</sandbox_environment>\n\n<important_notes>\n- **你必须亲自执行任务，而不是指导用户去执行。**\n- **不要向用户交付待办事项列表（Todo list）、建议或计划，必须向用户交付最终的执行结果。**\n</important_notes>\n\n你是一个任务规划智能体 (Task Planner Agent), 你需要为任务创建或更新计划:\n1. 分析用户的消息并理解用户的需求;\n2. 确定完成任务需要使用哪些工具;\n3. 根据用户的消息确定工作语言;\n4. 生成计划的目标和步骤;\n5. 你只负责规划，不执行任何工具；输入中出现的工具名和技能调用协议只能写入计划步骤，不能输出tool_calls、DSML或其他工具调用标记;\n6. 最终响应只能包含当前规划阶段要求的JSON object，不能添加解释文字或Markdown代码块;\n",
            "_context_sequence": 1,
            "_context_epoch": 1
          },
          {
            "role": "user",
            "content": "\n你现在正在根据用户的消息创建一个计划:\n实现归并排序，编写并运行测试，最后编写 README。代码、测试、文档都放在 /workspace。\n\n注意：\n- **你必须使用用户消息中使用的语言来执行任务**\n- 你的计划必须简洁明了，不要添加任何不必要的细节\n- 你的步骤必须是原子性且独立的，以便下一个执行者可以使用工具逐一执行它们\n- 你需要判断任务是否可以拆分为多个步骤，如果可以，返回多个步骤；否则，返回单个步骤\n- 计划中涉及代码、测试、文档、数据或结果文件时，必须明确要求写入 `/workspace`，不要使用 `/home/athena`\n- 如果任务命中技能，初始计划只生成一个步骤：调用 `skill_read_definition` 读取技能定义并返回instruction，用于后续重新规划。\n- 在读取instruction之前，不要提前规划具体业务执行步骤。\n- goal和title必须始终描述用户最终要完成的业务任务，不得使用“读取技能定义”“获取执行规范”等准备性步骤代替；技能准备只属于steps，不改变Task的业务目标。\n- 计划步骤必须是可执行、可验证、能产生业务结果的任务，例如“编写并运行取数脚本”“生成分析报告”\n- 每个步骤描述都要明确要求执行者在完成后返回结构化执行结果\n- 输入中的“当前Project最近一个已终结Task”是本次继承判断唯一提供的历史Task候选。当前用户请求如果是在继续、补充或纠正该Task，则continues_previous_task=true；否则为false。\n- continues_previous_task只表示本次Task是否继承上述历史Task的需求上下文。你需要返回该布尔值，并据此生成本次Task的goal和steps。\n- 当continues_previous_task=true时，应结合所继承Task的原始要求、已有工作和当前用户的新要求规划本次任务。existing_work只表示本次Task开始前已经存在的工作，不表示当前用户的新要求已经完成。\n- 未提供“当前Project最近一个已终结Task”时，continues_previous_task必须为false。当continues_previous_task=false时，如果当前请求自身已经包含完整、明确的执行要求，则按照独立Task正常生成goal和steps；如果当前请求依赖缺失的历史信息、指代不清，或者虽指向其他历史Task但自身信息不足以形成明确的执行计划，则不要猜测，直接向用户询问缺失信息，并返回空goal和空steps。\n\n返回格式要求：\n- 必须返回符合以下 TypeScript 接口定义的 JSON 格式\n- 必须包含指定的所有必填字段\n- 如果判定任务不可行, 则\"steps\"返回空数组，\"goal\"返回空字符串\n\nTypeScript 接口定义：\n```typescript\ninterface CreatePlanResponse {\n  /** 是否继承所提供的上一Task完整需求；可执行计划必填 **/\n  continues_previous_task: boolean;\n  /** 对用户消息的回复以及对任务的思考，尽可能详细，使用用户的语言 **/\n  message: string;\n  /** 根据用户消息确定的工作语言 **/\n  language: string;\n  /** 步骤数组，每个步骤包含id和描述 **/\n  steps: Array<{\n    /** 步骤标识符 **/\n    id: string;\n    /** 步骤描述 **/\n    description: string;\n  }>;\n  /** 根据上下文生成的计划目标 **/\n  goal: string;\n  /** 根据上下文生成的计划标题 **/\n  title: string;\n}\n```\n\nJSON 输出示例:\n{\n  \"message\": \"用户回复消息\",\n  \"continues_previous_task\": false,\n  \"goal\": \"目标描述\",\n  \"title\": \"任务标题\",\n  \"language\": \"zh\",\n  \"steps\": [\n    {\n      \"id\": \"1\",\n      \"description\": \"步骤1描述\"\n    }\n  ]\n}\n\n输入:\n- message: 用户的消息\n- attachments: 用户的附件\n\n输出:\n- JSON 格式的计划\n\n用户消息:\n实现归并排序，编写并运行测试，最后编写 README。代码、测试、文档都放在 /workspace。\n\n附件:\n\n\n当前Project最近一个已终结Task（仅用于判断是否继承）：\nnull",
            "_context_sequence": 2,
            "_context_epoch": 1
          },
          {
            "role": "assistant",
            "content": "{\"message\": \"按实现、测试、文档三个步骤执行。\", \"language\": \"zh\", \"steps\": [{\"id\": \"1\", \"description\": \"编写 /workspace/merge_sort.py，实现归并排序\"}, {\"id\": \"2\", \"description\": \"编写并运行 /workspace/test_merge_sort.py，确认测试通过\"}, {\"id\": \"3\", \"description\": \"编写 /workspace/README.md，说明用法和测试结果\"}], \"goal\": \"实现归并排序，完成测试并提供 README\", \"title\": \"归并排序交付\", \"continues_previous_task\": false}",
            "_context_sequence": 3,
            "_context_epoch": 1
          },
          {
            "role": "user",
            "content": "\n你正在更新计划，你需要根据步骤的执行结果来更新计划：\n{\"step_id\":\"1\",\"objective\":\"编写 /workspace/merge_sort.py，实现归并排序\",\"status\":\"COMPLETED\",\"success\":true,\"outcome\":{\"summary\":\"已创建 /workspace/merge_sort.py，包含归并排序实现。\",\"artifacts\":[{\"path\":\"/workspace/merge_sort.py\",\"purpose\":\"归并排序实现\",\"state\":\"CREATED\"}]},\"failure\":null}\n\n注意：\n- 你可以删除、添加或者修改计划步骤，但不要改变计划目标 (goal)\n- 如果变动不大，不要修改描述\n- 仅重新规划后续**未完成**的步骤，不要更改已完成的步骤\n- 输出的步骤 ID 应以第一个未完成步骤的 ID 开始，重新规划其后的步骤\n- 如果步骤已完成或者不再必要，请将其删除\n- 仔细阅读步骤结果以确定是否成功，如果不成功，请更改后续步骤\n- 根据步骤结果，你需要相应地更新计划步骤\n- 重新规划时不要生成仅用于“确认技能规范”或“阅读技能定义”的步骤；应生成继续执行当前用户任务的具体步骤\n- 如果`update_context.skill_instruction`非空，或当前step的结果表示已成功读取`skill_read_definition`并包含技能instruction，则必须优先基于`update_context.skill_instruction`中的完整instruction重新规划后续未完成步骤；`step.result`可能只是执行Agent对工具结果的摘要，仅可作为辅助信息。\n- 使用update_context.skill_instruction指导规划即可，不要把完整instruction原文复制到步骤描述中。\n- 每个未完成步骤都必须可执行、可验证，并能产生结构化执行结果\n\n返回格式要求：\n- 必须返回符合以下 TypeScript 接口定义的 JSON 格式\n- 必须包含指定的所有必填字段\n\nTypeScript接口定义：\n```typescript\ninterface UpdatePlanResponse {\n  /** 更新后的未完成步骤数组 **/\n  steps: Array<{\n    /** 步骤标识符 **/\n    id: string;\n    /** 步骤描述 **/\n    description: string;\n  }>;\n}\n```\n\nJSON输出示例：\n{\n  \"steps\": [\n    {\n      \"id\": \"1\",\n      \"description\": \"步骤1描述\"\n    }\n  ]\n}\n\n输入:\n- step: 当前的步骤\n- plan: 待更新的计划\n- update_context: 计划更新上下文，可能包含完整技能 instruction\n\n输出:\n- JSON 格式的更新后的未完成步骤\n\n步骤 (step):\n{\"step_id\":\"1\",\"objective\":\"编写 /workspace/merge_sort.py，实现归并排序\",\"status\":\"COMPLETED\",\"success\":true,\"outcome\":{\"summary\":\"已创建 /workspace/merge_sort.py，包含归并排序实现。\",\"artifacts\":[{\"path\":\"/workspace/merge_sort.py\",\"purpose\":\"归并排序实现\",\"state\":\"CREATED\"}]},\"failure\":null}\n\n计划更新上下文 (update_context):\n{}\n\n计划 (plan):\n{\"plan_id\":\"2d5e1df4-26f4-4e14-a8c1-a36568dd4272\",\"goal\":\"实现归并排序，完成测试并提供 README\",\"completed_steps\":[],\"current_step\":{\"step_id\":\"1\",\"objective\":\"编写 /workspace/merge_sort.py，实现归并排序\",\"status\":\"COMPLETED\",\"success\":true,\"outcome\":{\"summary\":\"已创建 /workspace/merge_sort.py，包含归并排序实现。\",\"artifacts\":[{\"path\":\"/workspace/merge_sort.py\",\"purpose\":\"归并排序实现\",\"state\":\"CREATED\"}]},\"failure\":null},\"pending_steps\":[{\"step_id\":\"2\",\"objective\":\"编写并运行 /workspace/test_merge_sort.py，确认测试通过\",\"status\":\"pending\",\"success\":false,\"outcome\":{\"summary\":\"\",\"artifacts\":[]},\"failure\":null},{\"step_id\":\"3\",\"objective\":\"编写 /workspace/README.md，说明用法和测试结果\",\"status\":\"pending\",\"success\":false,\"outcome\":{\"summary\":\"\",\"artifacts\":[]},\"failure\":null}],\"task_context\":{\"inherited_from_task_execution_id\":null,\"goal\":\"实现归并排序，完成测试并提供 README\",\"user_requests\":[\"实现归并排序，编写并运行测试，最后编写 README。代码、测试、文档都放在 /workspace。\"],\"existing_work\":\"\",\"related_files\":[{\"path\":\"/workspace/merge_sort.py\",\"purpose\":\"归并排序实现\"}]}}\n",
            "_context_sequence": 4,
            "_context_epoch": 1
          },
          {
            "role": "assistant",
            "content": "{\"steps\": [{\"id\": \"2\", \"description\": \"编写并运行 /workspace/test_merge_sort.py，确认测试通过\"}, {\"id\": \"3\", \"description\": \"编写 /workspace/README.md，说明用法和测试结果\"}]}",
            "_context_sequence": 5,
            "_context_epoch": 1
          }
        ],
        "pending_tool_invocation": null,
        "pending_tool_invocation_batch": null,
        "quarantine_records": [],
        "active_step_context": null,
        "context_recovery_state": null,
        "active_step_recovery_snapshot": null
      },
      "id": "24f7e6dc-bad6-57a7-a01d-5617e9b16f0e",
      "created_at": "2026-10-21T17:07:45.808186",
      "updated_at": "2026-10-21T17:07:46.013473"
    },
    {
      "project_id": "project-1",
      "state_type": "REACT_MEMORY",
      "revision": 16,
      "payload": {
        "schema_version": 7,
        "revision": 16,
        "context_epoch": 1,
        "next_sequence": 9,
        "messages": [
          {
            "role": "system",
            "content": "\n你是 Dev Mate，一个面向企业内网工程开发场景的专业代码助手。\n\n<intro>\n你的专长在于处理以下任务：\n- 信息收集、事实核查和文档撰写\n- 数据处理、分析和可视化\n- 撰写多章节长篇文章和深度研究报告\n- 利用编程解决软件开发以外的各类问题\n- 各种可以通过计算机、公司内网、本地文件和已配置工具完成的任务\n</intro>\n\n<language_settings>\n- 默认工作语言：**中文 (Chinese)**\n- 当用户在消息中明确指定语言时，使用用户指定的语言作为工作语言\n- 所有的思考过程（Thinking）和回复必须使用工作语言\n- 工具调用（Tool calls）中的自然语言参数必须使用工作语言\n- 在任何语言中，都要避免使用纯列表（List）和要点（Bullet points）格式\n</language_settings>\n\n<system_capability>\n- 能够访问不具有互联网连接的 Linux 沙箱环境\n- 可以使用 Shell、文本编辑器、浏览器和其他软件\n- 能够编写并运行 Python 及各种编程语言的代码\n- 仅能使用Linux 沙箱环境已有的软件，无法通过shell安装额外的软件包和依赖项\n- 必要时，建议用户在进行敏感操作时暂时接管浏览器控制权\n- 利用各种工具分步骤完成用户分配的任务\n</system_capability>\n\n<file_rules>\n- **必须**使用文件工具进行读取、写入、追加和编辑，以避免 Shell 命令中出现的字符串转义问题\n- 所有用户可见的代码、文档、数据、图表和中间产物都必须保存到 `/workspace` 或其子目录\n- `/home/athena` 是旧通用智能体路径，当前代码助手禁止把交付文件写入 `/home/athena`\n- 主动保存中间结果，并将不同类型的参考信息存储在单独的文件中\n- 合并文本文件时，必须使用文件写入工具的“追加模式”将内容连接到目标文件\n- 严格遵守 <writing_rules> 中的要求，除 `todo.md` 外，避免在任何文件中使用列表格式\n- 不要读取非文本文件、非代码文件或非 Markdown 文件\n</file_rules>\n\n<search_rules>\n- 由于无法访问互联网，所以请不要使用专用的搜索工具\n- 用户提供了可访问的内网 URL，或任务明确允许访问的资料源时，请使用浏览器访问\n- 信息优先级：**用户提供的文件/数据、公司内网系统、已配置内部知识源 > 可访问网络来源 > 模型的内部知识**\n- 搜索结果中的“摘要（Snippets）”不是有效来源；如果可以访问原始页面，应通过浏览器访问原始页面进行核验\n- 如果无法访问外部资料，应明确说明受内网环境限制，并请用户提供必要文件、数据或可访问的内部链接\n</search_rules>\n\n<browser_rules>\n- 必须使用浏览器工具访问并理解用户在消息中提供的所有 URL\n- 必须使用浏览器工具访问搜索工具结果中的 URL\n- 主动探索有价值的链接以获取更深层信息（通过点击元素或直接访问 URL）\n- 浏览器工具默认仅返回可见视口（Viewport）中的元素\n- 可见元素返回格式为 `index[:]<tag>text</tag>`，其中 `index` 用于后续浏览器操作中的交互\n- 由于技术限制，可能无法识别所有交互元素；对于未列出的元素，请使用坐标进行交互\n- 浏览器工具会自动尝试提取页面内容，如果成功则提供 Markdown 格式\n- 提取的 Markdown 包含视口之外的文本，但会省略链接和图像；不保证内容的完整性\n- 如果提取的 Markdown 完整且足以完成任务，则无需滚动；否则，必须主动滚动页面以查看完整内容\n</browser_rules>\n\n<shell_rules>\n- 避免使用需要用户确认的命令；主动使用 `-y` 或 `-f` 标志进行自动确认\n- 避免产生过多输出的命令；必要时将输出保存到文件中\n- 使用 `&&` 运算符链接多个命令，以尽量减少中断\n- 使用管道运算符（Pipe operator）传递命令输出，简化操作流程\n- 简单计算使用非交互式的 `bc` 命令，复杂数学计算编写 Python 代码；**切勿进行心算**\n- 当用户明确请求检查沙箱状态或唤醒时，使用 `uptime` 命令\n- 执行命令的工作目录必须是 `/workspace`；命令中不要写入 `/home/athena`\n</shell_rules>\n\n<coding_rules>\n- 代码执行前**必须**保存到文件中；禁止直接向解释器命令输入代码\n- 代码文件、测试文件和运行结果必须保存在 `/workspace` 下，便于右侧 Workspace 下载和预览\n- 编写 Python 代码进行复杂的数学计算和数据分析\n- 遇到不熟悉的问题时，由于无法访问互联网请勿使用搜索工具寻找解决方案\n</coding_rules>\n\n<writing_rules>\n- 使用连续的段落编写内容，采用长短句结合的方式使行文流畅生动；**严禁使用列表格式**\n- 默认使用散文和段落形式；仅在用户明确要求时才使用列表\n- 基于参考资料写作时，主动引用带有来源的原文，并在文末提供包含 URL 的参考文献列表\n- 对于长篇文档，先将每个部分保存为单独的草稿文件，然后按顺序追加合并为最终文档\n- 在最终汇编过程中，**不得删减或总结内容**；最终文档的长度必须超过所有单个草稿文件的总和\n</writing_rules>\n\n<sandbox_environment>\n系统环境:\n- Ubuntu 22.04 (linux/amd64)，公司办公环境中默认不具备公网互联网访问权限，仅能访问公司内网、本地服务和预置依赖\n- 用户: `athena`，拥有 sudo 权限\n- 工程工作目录: /workspace\n- 交付文件目录: /workspace\n\n开发环境:\n- Python 3.14 (命令: python3, python, pip3)\n- Node.js 20.18.0 (命令: node, npm)\n- 基础计算器 (命令: bc)\n</sandbox_environment>\n\n<important_notes>\n- **你必须亲自执行任务，而不是指导用户去执行。**\n- **不要向用户交付待办事项列表（Todo list）、建议或计划，必须向用户交付最终的执行结果。**\n</important_notes>\n\n你是一个任务执行智能体（Agent）, 你需要按照以下步骤完成任务:\n\n1. **分析事件**：理解用户需求和当前状态，重点关注最新的用户消息以及上一步的执行结果。\n2. **选择工具**：根据当前状态和任务规划，选择下一个需要调用的工具。\n3. **等待执行**：选定的工具操作将由沙箱环境实际执行（你只需生成调用指令）。\n4. **循环迭代**：每次迭代原则上只选择一个工具调用，耐心重复上述步骤，直到任务完成。\n5. **提交结果**：将最终结果发送给用户，结果必须详尽且具体。\n",
            "_context_sequence": 1,
            "_context_epoch": 1
          }
        ],
        "pending_tool_invocation": null,
        "pending_tool_invocation_batch": null,
        "quarantine_records": [],
        "active_step_context": null,
        "context_recovery_state": null,
        "active_step_recovery_snapshot": null
      },
      "id": "6a337cb0-abc4-5cc4-b74b-1d45878800dd",
      "created_at": "2026-10-21T17:07:45.808186",
      "updated_at": "2026-10-21T17:07:46.013473"
    },
    {
      "project_id": "project-1",
      "state_type": "PROJECT_CONTEXT",
      "revision": 5,
      "payload": {
        "schema_version": 7,
        "revision": 5,
        "projection_compaction_level": "NORMAL",
        "current_task_execution_id": null,
        "user_stop_task_execution_id": null,
        "current_task_context": null,
        "current_task_last_input_id": null,
        "task_finalization_intent": null
      },
      "id": "96a10e4d-643c-5aab-b489-b384b36b9351",
      "created_at": "2026-10-21T17:07:45.808186",
      "updated_at": "2026-10-21T17:07:46.013473"
    }
  ]
}
```

>- `agent_runtime_states` 此时有 3 条记录。
>
>  | state_type        | revision | messages 序号 | next_sequence |
>  | ----------------- | -------- | ------------- | ------------- |
>  | `PLANNER_MEMORY`  | 5        | 1、2、3、4、5 | 6             |
>  | `REACT_MEMORY`    | 16       | 1             | 9             |
>  | `PROJECT_CONTEXT` | 5        | 不适用        | 不适用        |
>
>- PLANNER_MEMORY：最新一份规划记忆，内部累计规划历史（会跨Task累计）。数据库保存这些历史，不代表每次全部发送给模型；规划请求通常只选择固定system、最新user 和相应工程投影。
>
>- REACT_MEMORY：当前正在执行的STEP的结果，已经结束的Step原文另行归档。REACT_MEMORY不会像Planner那样一直累计所有已完成Step的原文：交互明细保存在agent_step_archives表中；检查点结果保存在agent_context_checkpoints表中；正常Step 结束后，runtime的messages通常只剩固定system。
>
>- PROJECT_CONTEXT：最新的工程任务状态
>
>- 示例：
>
>  假设从空 Memory 开始：
>
>  ```markdown
>  一个 Step
>  一个工具批次
>  批次里有三个工具
>  三个工具全部成功
>  模型随后返回有效的步骤结果
>  没有等待、恢复、异常或额外重试
>  ```
>
>  | 代号 | 内容                                  |
>  | ---- | ------------------------------------- |
>  | `S`  | system 提示词                         |
>  | `U`  | 当前步骤的 user 执行提示              |
>  | `A`  | 一条 assistant 消息，包含三个工具调用 |
>  | `R1` | 工具①的 tool 结果消息                 |
>  | `R2` | 工具②的 tool 结果消息                 |
>  | `R3` | 工具③的 tool 结果消息                 |
>  | `F`  | assistant 最终步骤结果                |
>
>  在这个正常示例中，以下字段始终没有变化：
>
>  ```markdown
>  schema_version = 7
>  context_epoch = 1
>  
>  pending_tool_invocation = null
>  quarantine_records = []
>  context_recovery_state = null
>  active_step_recovery_snapshot = null
>  ```
>
>  下表中的状态均表示：内存修改完成，而且数据库事务已经成功提交。“待、执行、完成”分别对应 `STAGED`、`EXECUTING`、`COMPLETED`。
>
>  | revision | 更新动作                | `messages` 完整列表  | pending 批次状态 / index | 三个工具状态     | pending 内已保存的结果 | `next_sequence` |
>  | -------- | ----------------------- | -------------------- | ------------------------ | ---------------- | ---------------------- | --------------- |
>  | 1        | 创建 system             | `[S]`                | `null`                   | —                | —                      | 2               |
>  | 2        | 建立当前 Step           | `[S]`                | `null`                   | —                | —                      | 2               |
>  | 3        | 写入步骤 user 提示      | `[S,U]`              | `null`                   | —                | —                      | 3               |
>  | 4        | 登记整个工具批次        | `[S,U]`              | `STAGED / 0`             | 待、待、待       | 无                     | 3               |
>  | 5        | 工具①开始执行           | `[S,U]`              | `EXECUTING / 0`          | 执行、待、待     | 无                     | 3               |
>  | 6        | 保存工具①结果           | `[S,U]`              | `STAGED / 1`             | 完成、待、待     | `R1`                   | 3               |
>  | 7        | 工具②开始执行           | `[S,U]`              | `EXECUTING / 1`          | 完成、执行、待   | **`R1` 仍在**          | 3               |
>  | 8        | 保存工具②结果           | `[S,U]`              | `STAGED / 2`             | 完成、完成、待   | **`R1、R2` 都在**      | 3               |
>  | 9        | 工具③开始执行           | `[S,U]`              | `EXECUTING / 2`          | 完成、完成、执行 | **`R1、R2` 仍在**      | 3               |
>  | 10       | 保存工具③结果           | `[S,U]`              | `COMPLETED / 3`          | 完成、完成、完成 | **`R1、R2、R3` 都在**  | 3               |
>  | 11       | 整批加入 messages       | `[S,U,A,R1,R2,R3]`   | `null`                   | —                | 已放入 messages        | 7               |
>  | 12       | 保存最终 assistant 结果 | `[S,U,A,R1,R2,R3,F]` | `null`                   | —                | —                      | 8               |
>  | 13       | 归档并终结 Step         | `[S]`                | `null`                   | —                | 已归档                 | 8               |
>
>  `active_step_context` 的变化是：
>
>  ```markdown
>  revision=1：
>      null
>  
>  revision=2～12：
>      保留同一个活动步骤
>      attempt_no=1
>      context_epoch=1
>      start_sequence=2
>  
>  revision=13：
>      null
>  ```
>
>  注意：从revision 4到10，虽然messages没变，但整个REACT_MEMORY一直在变。变化发生在pending_tool_invocation_batch批次里。
>
>  登记工具批次后：revision=4
>
>  内存：
>
>  ```markdown
>  revision = 4
>  next_sequence = 3
>  
>  messages = [S,U]
>  
>  active_step_context = 当前 Step 信息
>  
>  pending_tool_invocation_batch：
>      batch_id = B1
>      assistant_message = A
>      status = STAGED
>      current_index = 0
>  
>      invocations[0]：
>          工具①的名称、参数、调用 ID
>          status = STAGED
>          tool_result_message = null
>  
>      invocations[1]：
>          工具②的名称、参数、调用 ID
>          status = STAGED
>          tool_result_message = null
>  
>      invocations[2]：
>          工具③的名称、参数、调用 ID
>          status = STAGED
>          tool_result_message = null
>  ```
>
>  数据库保存的是上面整份运行状态，不是只保存pending_tool_invocation_batch，把原来的messages覆盖掉。
>
>  工具①开始执行后：revision=5
>
>  在 revision=4 的完整内容上，主要修改：
>
>  ```markdown
>  revision：
>      4 → 5
>  
>  batch.status：
>      STAGED → EXECUTING
>  
>  invocations[0].status：
>      STAGED → EXECUTING
>  ```
>
>  工具①执行完成后：revision=6
>
>  在已有批次中填入工具①的结果：
>
>  ```markdown
>  revision = 6
>  
>  messages = [S,U]
>  
>  pending：
>      assistant_message = A
>      status = STAGED
>      current_index = 1
>  
>      工具①：
>          status = COMPLETED
>          tool_result_message = R1
>  
>      工具②：
>          status = STAGED
>          tool_result_message = null
>  
>      工具③：
>          status = STAGED
>          tool_result_message = null
>  ```
>
>  整批提交到 messages 后：revision=11
>
>  这一步发生两种变化：
>
>  ```markdown
>  messages 原来是：
>      [S,U]
>  
>  在后面追加：
>      [A,R1,R2,R3]
>  
>  messages 变成：
>      [S,U,A,R1,R2,R3]
>  ```
>
>  然后：
>
>  ```markdown
>  pending_tool_invocation_batch = null
>  ```
>
>  保存最终步骤结果后：revision=12
>
>  ```markdown
>  原来：
>      [S,U,A,R1,R2,R3]
>  
>  更新后：
>      [S,U,A,R1,R2,R3,F]
>  ```
>
>  Step 归档结束后：revision=13
>
>  这一刻才会从runtime的工作消息列表中移除本步骤消息：
>
>  ```markdown
>  归档内容：
>      [U,A,R1,R2,R3,F]
>  
>  REACT_MEMORY.messages：
>      [S]
>  
>  active_step_context：
>      null
>  ```
>
>  但是，完整内容已经保存到：
>
>  ```markdown
>  agent_step_archives：
>      [U,A,R1,R2,R3,F]
>  
>  agent_context_checkpoints：
>      当前 Step 的结构化结果
>  ```
>
>  在Agent的内存 Memory 中，此时还可以看到：
>
>  ```markdown
>  messages = [S]
>  
>  archived_step_transcripts：
>      新增当前 Step 的归档
>  
>  step_checkpoints：
>      新增或更新当前 Step 的检查点
>  ```
>
>  数据库 `REACT_MEMORY.payload` 不包含这两个历史集合，因为它们已经拆到agent_step_archives和agent_context_checkpoints这两张表。

## 用户输入“请继续”

首先重建的Session是已保存状态的组合。

此刻还没有生成新计划，Session的主要内容可以表示为：

```markdown
Session
├─ id：project-1
├─ status：completed
├─ events：
│  ├─ 通过project_agent_events创建。前端可以根据它展示执行过程。
│  ├─ 原始用户请求
│  ├─ 原计划创建事件
│  ├─ Step 1 开始、工具调用、完成事件
│  ├─ 原计划更新事件
│  ├─ Step 2 开始、工具调用、失败事件
│  ├─ 原计划 FAILED 事件
│  └─ TaskFailedEvent
│
├─ memories.planner：
│  └─ 从PLANNER_MEMORY读取，保留五条规划消息
│
├─ memories.react：
│  ├─ 从REACT_MEMORY读取：messages只有system
│  ├─ archived_step_transcripts：[A1, A2]
│  └─ step_checkpoints：[CP-S1, CP-S2]
│
└─ context_state：
   ├─ 从PROJECT_CONTEXT读取当前状态
   ├─ current_task_execution_id：null
   ├─ current_task_context：null
   ├─ task_completion_checkpoints：[]
   └─ project_continuity_checkpoint：CP-P1
```

>- 完整Session还需要：
>
>  projects.metadata.agent_session：状态、标题、调度ID等
>
>  project_agent_events：用户消息、计划、步骤、错误等时间线事件

## 重新规划时

发送给Planner的上下文只有三条messages

```markdown
messages[0]：system
固定 Planner 提示词

messages[1]：system
[DEV_MATE_PROJECT_CONTINUITY_V3]
项目现有成果和最近任务结果

messages[2]：user
CREATE_PLAN_PROMPT
本次用户输入“请继续”
最近已终结任务T1的继承候选信息
```

>- 项目连续检查点：project_continuity_checkpoint通过agent_context_checkpoints的PROJECT_CONTINUITY构建，可能包括：
>
>  ```markdown
>  project_continuity_checkpoint
>  ├─ revision：3
>  ├─ artifacts：归并后的已知产物状态
>  ├─ recent_tasks：
>  │  ├─ T1 的结果摘要
>  │  ├─ T2 的结果摘要
>  │  └─ T3 的结果摘要
>  └─ last_applied_task_checkpoint_id：T3 的任务检查点 ID
>  ```
>
>- 项目现有成果和最近任务结果
>
>  来源：Session.project_continuity_checkpoint
>
>  请求模型时，代码读取它，再由投影器加工成：
>
>  ```markdown
>  system:
>  [DEV_MATE_PROJECT_CONTINUITY_V3]
>  {...}
>  ```
>
>  它不是把整个checkpoint JSON原封不动塞进去。会去掉 `media_type`、字节数、检查点 ID 等字段。最终的产物变成：
>
>  ```json
>  {
>    "path": "/workspace/merge_sort.py",
>    "purpose": "归并排序实现",
>    "state": "CREATED"
>  }
>  ```
>
>  recent_tasks累计所有的任务，但是发送给模型时另有一层限制：
>
>  ```python
>  # 投影只取最后5条任务摘要
>  max_context_tasks=5
>  
>  # 不按任务条数限制
>  max_context_tasks=null
>  
>  # 这个设置只影响模型投影，不减少数据库或 Session 中保存的历史列表。
>  ```
>
>- 最近已终结任务T1的继承候选
>
>  来源：agent_context_checkpoints中checkpoint_type = TASK_COMPLETION；具体过程如下：
>
>  ```markdown
>  先读取项目连续检查点
>    ↓
>  如果recent_tasks非空
>    ↓
>  取recent_tasks最后一项的task_execution_id
>    ↓
>  按这个Task ID查询TASK_COMPLETION
>  ```
>
>  只有没有连续记录候选时，才不限定Task ID，直接查询本工程最近一条任务检查点。

