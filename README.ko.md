# Notion MCP 서버

> [!NOTE] 
> 
> 원격 MCP 서버인 **Notion MCP**(베타)가 다음과 같은 개선 사항과 함께 도입되었습니다:
> - 표준 OAuth를 통한 간편한 설치. 더 이상 JSON이나 API 토큰을 직접 다룰 필요가 없습니다.
> - AI 에이전트에 최적화된 강력한 도구들. 이 도구들은 토큰 소비를 최적화하도록 설계되었습니다.
> 
> [여기](https://notion.notion.site/Beta-Overview-Notion-MCP-206efdeead058060a59bf2c14202bd0a)에서 더 자세히 알아보고 사용해 보세요.


![notion-mcp-sm](https://github.com/user-attachments/assets/6c07003c-8455-4636-b298-d60ffdf46cd8)

이 프로젝트는 [Notion API](https://developers.notion.com/reference/intro)를 위한 [MCP 서버](https://spec.modelcontextprotocol.io/)를 구현합니다.

![mcp-demo](https://github.com/user-attachments/assets/e3ff90a7-7801-48a9-b807-f7dd47f0d3d6)

### 설치

#### 1. Notion에서 통합 설정하기:
[https://www.notion.so/profile/integrations](https://www.notion.so/profile/integrations)로 이동하여 새로운 **내부** 통합을 생성하거나 기존 통합을 선택하세요.

![Notion 통합 토큰 생성](docs/images/integrations-creation.png)

저희가 노출하는 Notion API의 범위를 제한하지만(예: MCP를 통해 데이터베이스를 삭제할 수 없음), LLM에 작업 공간 데이터를 노출하는 데에는 약간의 위험이 따릅니다. 보안에 민감한 사용자는 통합의 _기능_을 추가로 구성할 수 있습니다.

예를 들어, "구성" 탭에서 "콘텐츠 읽기" 접근 권한만 부여하여 읽기 전용 통합 토큰을 생성할 수 있습니다:

![읽기 콘텐츠가 체크된 Notion 통합 토큰 기능](docs/images/integrations-capabilities.png)

#### 2. 통합에 콘텐츠 연결하기:
관련 페이지와 데이터베이스가 통합에 연결되었는지 확인하세요.

이를 위해 내부 통합 설정의 **액세스** 탭을 방문하세요. 액세스 권한을 편집하고 사용하려는 페이지를 선택합니다.
![통합 액세스 탭](docs/images/integration-access.png)

![통합 액세스 편집](docs/images/page-access-edit.png)

또는, 개별적으로 페이지 접근 권한을 부여할 수도 있습니다. 대상 페이지를 방문하여 점 3개를 클릭하고 "통합에 연결"을 선택해야 합니다.

![Notion 연결에 통합 토큰 추가](docs/images/connections.png)

#### 3. 클라이언트에 MCP 설정 추가하기:

##### npm 사용:

**Cursor와 Claude:**

다음 내용을 `.cursor/mcp.json` 또는 `claude_desktop_config.json`(MacOS: `~/Library/Application\ Support/Claude/claude_desktop_config.json`)에 추가하세요.

```javascript
{
  "mcpServers": {
    "notionApi": {
      "command": "npx",
      "args": ["-y", "@notionhq/notion-mcp-server"],
      "env": {
        "OPENAPI_MCP_HEADERS": "{\"Authorization\": \"Bearer ntn_****\", \"Notion-Version\": \"2022-06-28\" }"
      }
    }
  }
}
```

**Zed**

다음 내용을 `settings.json`에 추가하세요.

```json
{
  "context_servers": {
    "some-context-server": {
      "command": {
        "path": "npx",
        "args": ["-y", "@notionhq/notion-mcp-server"],
        "env": {
          "OPENAPI_MCP_HEADERS": "{\"Authorization\": \"Bearer ntn_****\", \"Notion-Version\": \"2022-06-28\" }"
        }
      },
      "settings": {}
    }
  }
}
```

##### Docker 사용:

Docker로 MCP 서버를 실행하는 두 가지 옵션이 있습니다:

###### 옵션 1: 공식 Docker Hub 이미지 사용:

다음 내용을 `.cursor/mcp.json` 또는 `claude_desktop_config.json`에 추가하세요:

```javascript
{
  "mcpServers": {
    "notionApi": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "-e", "OPENAPI_MCP_HEADERS",
        "mcp/notion"
      ],
      "env": {
        "OPENAPI_MCP_HEADERS": "{\"Authorization\":\"Bearer ntn_****\",\"Notion-Version\":\"2022-06-28\"}"
      }
    }
  }
}
```

이 방법은:
- 공식 Docker Hub 이미지를 사용합니다.
- 환경 변수를 통해 JSON 이스케이프를 올바르게 처리합니다.
- 더 신뢰할 수 있는 구성 방법을 제공합니다.

###### 옵션 2: 로컬에서 Docker 이미지 빌드:

로컬에서 Docker 이미지를 빌드하고 실행할 수도 있습니다. 먼저, Docker 이미지를 빌드하세요:

```bash
docker-compose build
```

그런 다음, 다음 내용을 `.cursor/mcp.json` 또는 `claude_desktop_config.json`에 추가하세요:

```javascript
{
  "mcpServers": {
    "notionApi": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "-e",
        "OPENAPI_MCP_HEADERS={\"Authorization\": \"Bearer ntn_****\", \"Notion-Version\": \"2022-06-28\"}",
        "notion-mcp-server"
      ]
    }
  }
}
```

`ntn_****`을 당신의 통합 시크릿으로 교체하는 것을 잊지 마세요. 통합 구성 탭에서 찾을 수 있습니다:

![개발자 포털의 구성 탭에서 통합 토큰 복사](https://github.com/user-attachments/assets/67b44536-5333-49fa-809c-59581bf5370a)


#### Smithery를 통한 설치

[![smithery badge](https://smithery.ai/badge/@makenotion/notion-mcp-server)](https://smithery.ai/server/@makenotion/notion-mcp-server)

[Smithery](https://smithery.ai/server/@makenotion/notion-mcp-server)를 통해 Claude 데스크톱용 Notion API 서버를 자동으로 설치하려면:

```bash
npx -y @smithery/cli install @makenotion/notion-mcp-server --client claude
```

### 예시

1. 다음 지시어 사용
```
"Getting started" 페이지에 "Hello MCP"라고 댓글 달기
```

AI는 작업을 완수하기 위해 `v1/search`와 `v1/comments`라는 두 개의 API 호출을 올바르게 계획할 것입니다.

2. 유사하게, 다음 지시어는 "Development" 부모 페이지에 "Notion MCP"라는 이름의 새 페이지를 추가하는 결과를 낳습니다.
```
"Development" 페이지에 "Notion MCP"라는 제목의 페이지 추가하기
```

3. 콘텐츠 ID를 직접 참조할 수도 있습니다.
```
페이지 1a6b35e6e67f802fa7e1d27686f017f2의 콘텐츠 가져오기
```

### 개발

빌드

```
npm run build
```

실행

```
npx -y --prefix /path/to/local/notion-mcp-server @notionhq/notion-mcp-server
```

배포

```
npm publish --access public
```