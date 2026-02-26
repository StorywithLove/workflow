# Realtime Update
**项目来源**：https://github.com/HeQinWill/CNEMC   
**Realtime Update** 是一个自动触发CNEMC(目标为10分钟Himawari SWR)更新的 GitHub Workflow。由于 GitHub Actions 的 **cron 定时触发不稳定**，通过简单的 Python 脚本，通过 **GitHub REST API 的 [workflow dispatch](https://docs.github.com/en/rest/actions/workflows?apiVersion=2022-11-28#create-a-workflow-dispatch-event) 接口**远程触发 workflow。

示例代码：

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
