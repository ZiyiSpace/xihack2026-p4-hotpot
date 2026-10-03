# 下一盘 · 旋转小火锅多站点感知与智能补菜

> ⚠️ **本仓库已停止维护（2026-10-03）**：全队主仓库迁移至 **https://github.com/ZiyiSpace/xihack2026-p4-next-plate**，后端已归位至其 `modules/backend/`。此处仅作历史存档，请勿在此继续提交。

XiHack2026 企业赛道四 · 西安红考拉旋转小火锅。

**这个仓库是什么**：旋转小火锅门店的感知与补菜系统——采集端上报"盘子经过时的图片 + 称重"，后端解释每盘菜发生了什么、还剩多少、什么时候补、哪个菜受欢迎、浪费在哪。

## 目录结构

```
backend/               后端（FastAPI，雏形已可跑）
  app/
    core.py            服务编排（HTTP API 与回放共用一条代码路径）
    interpreter.py     确定性事件解释状态机（取用/补菜/尖峰异常）
    analytics.py       速度/预计耗尽/滞留/偏好/报损/经营分析
    refill.py          规则补菜引擎（任务生成/合并/闭环）
    jev.py             Jev 视觉服务器客户端（限并发+重试）
    vision.py          视觉适配层（认菜校验/菜量交叉验证，失败自动降级）
    replay.py          数据集回放 + answer_key 对评
  static/index.html    极简看板（盘子/菜品/任务三视图）
  README.md            后端详细文档：接口契约、启动、实测结论
hotpot_dataset_v0_1/   合成测试数据集（2 盘 / 8 观测 / 场景真值）
```

## 快速启动

```bash
cd backend
cp .env.example .env     # 填 JEV_API_KEY；留空 = 离线规则模式
pip install -r requirements.txt
uvicorn app.main:app --port 8000
```

- 看板：http://127.0.0.1:8000/ （点「回放数据集」一键演示）
- API 文档：http://127.0.0.1:8000/docs

## 模块对接（见 backend/README.md 的接口契约）

| 模块 | 负责人 | 对接接口 |
|---|---|---|
| 站点采集 | 岳浩宇 | `POST /api/events/station/upload`（图+表单） |
| 前端/集成 | 力一雄 | `GET /api/plates` `/api/dishes` `/api/tasks` + 任务操作 |
| 视觉模块 | 子怡 | `backend/app/vision.py`（已在 Jev 服务器上验证） |
| 需求/验收 | 逐月者 | `backend/README.md` 规则章节 |

## 状态（2026-10-03）

- 数据集回放离线/在线均 **8/8** 事件分类与场景设定一致
- 认菜校验 **6/6**（置信度 0.83–0.99）；菜量以称重规则为准，模型仅旁证
- 补菜任务生成/闭环、撤盘报损、换菜不串盘、服务重启状态恢复均已验证
