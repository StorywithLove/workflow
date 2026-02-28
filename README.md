# 实时数据爬取与存档

本项目用于 **自动化爬取并存档实时环境数据**，并通过 **GitHub Actions + 外部定时触发机制** 实现稳定、可持续的周期性更新，适用于对 **时效性与连续性要求较高** 的数据采集场景。

## 项目背景

**项目来源**  
- 原始数据结构与实现思路参考自：  
  👉 https://github.com/HeQinWill/CNEMC

在此基础上，本项目对 **调度方式与稳定性** 进行了增强。

## 功能概述

一个基于 **GitHub Actions Workflow** 的自动化更新流程，目前支持 / 规划支持：

- 🌏 **CNEMC 实时环境监测数据更新**
- 🛰️ **10 分钟级 Himawari 卫星短波辐射（SWR）数据**（规划中）

为避免 GitHub Actions 原生 `cron` 定时存在的触发不稳定问题，本项目采用 **外部定时触发 Workflow 的方式**，提升长期运行的可靠性。

---

## 配置流程

### 1. 项目 Token 配置

在 **项目仓库（repo）** 中依次进入：  
Settings → Secrets and variables → Actions → Secrets → New repository secret  
新增一个仓库级环境变量：

- **Name**：`GTOKEN`
- **Value**：`<your_personal_access_token>`

---

### 2. Personal Access Token（PAT）创建

在 **个人 GitHub 账号** 中依次进入：
在 **个人 GitHub 账号** 中依次进入：
Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token (classic)
创建 Token 时请注意：

- **Token 类型**：`classic`
- **Scopes 必须勾选**：
  - `repo`
  - `workflow`

> ⚠️ 若未勾选 `workflow` 权限，将无法通过 API 触发 `workflow_dispatch`。

---


### 3. 外部定时触发（推荐）

GitHub Actions 自带的 `cron` 定时触发在实际运行中 **存在延迟或漏触发的问题**，因此本项目推荐使用：

> **云函数 + GitHub REST API 远程触发 Workflow**

#### 触发方式说明

通过云函数定时调用 GitHub 官方提供的 **Workflow Dispatch 接口**：

- 📘 官方文档：  
  👉 [Create a workflow dispatch event](https://docs.github.com/en/rest/actions/workflows?apiVersion=2022-11-28#create-a-workflow-dispatch-event)

该方式可显著提升定时任务的 **稳定性与可控性**。

---

### 4. 云函数平台选择

当前已验证可用的平台：

- ☁️ [腾讯云函数 SCF](https://cloud.tencent.com/document/product/583/113039)

理论上，任何支持 **HTTP 请求 + 定时触发** 的云函数平台均可采用类似方式部署。

---

### 5. 云函数部署注意事项（重要）

在云函数部署与调试过程中，请注意以下事项：

- **将配置参数直接写为字符串常量**
- **避免通过 `event` / `context` 等方式传递关键参数**

这样可以有效规避因平台参数传递机制导致的：

- 参数丢失  
- 环境变量解析失败  
- 非预期的运行异常  

> ✅ 建议做法：在代码中直接硬编码  
> `repo_owner`、`repo_name`、`workflow_name` 等关键参数。

---

## 示例代码（云函数触发 Workflow）

```python
import requests
import json

def run(github_pat, repo_owner, repo_name='workflow', workflow_name='realtime.yml'):
    payload = json.dumps({"ref": "main"})
    headers = {
        'Authorization': f'Bearer {github_pat}',
        "Accept": "application/vnd.github.v3+json"
    }
    url = f'https://api.github.com/repos/{repo_owner}/{repo_name}/actions/workflows/{workflow_name}/dispatches'
    response = requests.post(url, data=payload, headers=headers)
    assert response.status_code == 204, f"Failed to trigger workflow: {response.text}"
