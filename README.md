# Azure Openai Proxy

Azure OpenAI Service Proxy，能够将 Azure OpenAI API 服务转换为 OpenAI API 兼容的请求，支持 GPT-4、GPT-3.5、Embeddings。

修改自[stulzq/azure-openai-proxy](https://github.com/stulzq/azure-openai-proxy)。

## 验证可用的开源项目

| Name                                                     | Status |
| -------------------------------------------------------- | ------ |
| [chatgpt-web](https://github.com/Chanzhaoyu/chatgpt-web) | √   |
| [chatbox](https://github.com/Bin-Huang/chatbox)          | √    |
| [langchain](https://python.langchain.com/en/latest/)     | √    |
| [ChatGPT-Next-Web](https://github.com/Yidadaa/ChatGPT-Next-Web) | √ |


## 下载镜像

```bash
docker pull soulteary/azure-openai-proxy:1.2.0
```

## 快速上手

```yaml
version: "3.4"

services:
  azure-openai-proxy:
    image: stulzq/azure-openai-proxy:v1.1.0
    restart: always
    ports:
      - "8080:8080"
    # volumes:
    # You can also use configuration files
    # - ./config.yaml:/app/config.yaml
    environment:
      - TZ=Asia/Shanghai
      # 使用以 `https://` 开头，以 `.openai.azure.com/` 结尾的 URL
      - AZURE_OPENAI_ENDPOINT=https://<Azure OpenAI Endpoint Name>.openai.azure.com/
      # 你需要使用映射字符串，来将程序调用的模型，和你实际 Azure 部署的模型关联起来: gpt-3.5-turbo:gpt-35-turbo
      - AZURE_OPENAI_MODEL_MAPPER=gpt-3.5-turbo:gpt-35-turbo
      # 如果你定义了这个参数，那么你的程序在调用 “OpenAI API” 的时候，就不需要携带 KEY 的内容了，更方便，也更安全
      - AZURE_OPENAI_API_KEY=<Azure OpenAI API Key>
      # 推荐你始终使用 Stable 版本的 API，而非 Preview
      - AZURE_OPENAI_API_VER=2023-05-15
```

## 原理

![aoai-proxy.jpg](assets/images/aoai-proxy.jpg)

## 网络代理

如果你的网络不能够直接访问 Azure 云服务器，可以考虑设置下面的参数。

**HTTP Proxy**

Env:

````shell
AZURE_OPENAI_HTTP_PROXY=http://127.0.0.1:1087
````



**Socks5 Proxy**

Env:

````shell
AZURE_OPENAI_SOCKS_PROXY=socks5://127.0.0.1:1080
````

## 调用验证

使用 Curl ：

````shell
curl --location --request POST 'localhost:8080/v1/chat/completions' \
-H 'Authorization: Bearer <Azure OpenAI Key>' \
-H 'Content-Type: application/json' \
-d '{
    "max_tokens": 1000,
    "model": "gpt-3.5-turbo",
    "temperature": 0.8,
    "top_p": 1,
    "presence_penalty": 1,
    "messages": [
        {
            "role": "user",
            "content": "Hello"
        }
    ],
    "stream": true
}'
````

### 使用 ChatGPT-Next-Web

![chatgpt-web](assets/images/chatgpt-next-web.png)

docker-compose.yml

````yaml
version: '3'

services:
  chatgpt-web:
    image: yidadaa/chatgpt-next-web
    ports:
      - 3000:3000
    environment:
      OPENAI_API_KEY: <Azure OpenAI API Key>
      BASE_URL: http://azure-openai:8080
      CODE: ""
      HIDE_USER_API_KEY: 1
      HIDE_BALANCE_QUERY: 1
    depends_on:
      - azure-openai
    links:
      - azure-openai
    networks:
      - chatgpt-ns

  azure-openai:
    image: soulteary/azure-openai-proxy:1.2.0
    ports:
      - 8080:8080
    environment:
      AZURE_OPENAI_ENDPOINT: <Azure OpenAI API Endpoint>
      AZURE_OPENAI_MODEL_MAPPER: <Azure OpenAI API Deployment Mapper>
      AZURE_OPENAI_MODEL_MAPPER: gpt-4:gpt-4,gpt-3.5-turbo:gpt-35-turbo
      AZURE_OPENAI_API_VER: 2023-07-01-preview
    networks:
      - chatgpt-ns

networks:
  chatgpt-ns:
    driver: bridge
````

### 使用 ChatGPT-Web

ChatGPT Web: https://github.com/Chanzhaoyu/chatgpt-web

![chatgpt-web](assets/images/chatgpt-web.png)

Envs:

- `OPENAI_API_KEY` Azure OpenAI API Key
- `AZURE_OPENAI_ENDPOINT` Azure OpenAI API Endpoint
- `AZURE_OPENAI_MODEL_MAPPER` Azure OpenAI API Deployment Name Mappings

docker-compose.yml:

````yaml
version: '3'

services:
  chatgpt-web:
    image: chenzhaoyu94/chatgpt-web
    ports:
      - 3002:3002
    environment:
      OPENAI_API_KEY: <Azure OpenAI API Key>
      OPENAI_API_BASE_URL: http://azure-openai:8080
      # OPENAI_API_MODEL: gpt-4
      AUTH_SECRET_KEY: ""
      MAX_REQUEST_PER_HOUR: 1000
      TIMEOUT_MS: 60000
    depends_on:
      - azure-openai
    links:
      - azure-openai
    networks:
      - chatgpt-ns

  azure-openai:
    image: soulteary/azure-openai-proxy:1.2.0
    ports:
      - 8080:8080
    environment:
      AZURE_OPENAI_ENDPOINT: <Azure OpenAI API Endpoint>
      AZURE_OPENAI_MODEL_MAPPER: <Azure OpenAI API Deployment Mapper>
      AZURE_OPENAI_API_VER: 2023-07-01-preview
    networks:
      - chatgpt-ns

networks:
  chatgpt-ns:
    driver: bridge
````

### 使用配置文件

config.yaml

````yaml
api_base: "/v1"
deployment_config:
  - deployment_name: "xxx"
    model_name: "text-davinci-003"
    endpoint: "https://xxx-east-us.openai.azure.com/"
    api_key: "11111111111"
    api_version: "2023-03-15-preview"
  - deployment_name: "yyy"
    model_name: "gpt-3.5-turbo"
    endpoint: "https://yyy.openai.azure.com/"
    api_key: "11111111111"
    api_version: "2023-03-15-preview"
  - deployment_name: "zzzz"
    model_name: "text-embedding-ada-002"
    endpoint: "https://zzzz.openai.azure.com/"
    api_key: "11111111111"
    api_version: "2023-03-15-preview"
````

程序默认读取配置文件为 `<workdir>/config.yaml`, 你可以使用参数调整读取配置文件的路径 `-c config.yaml`。

docker-compose:

````yaml
azure-openai:
    image: stulzq/azure-openai-proxy
    ports:
      - 8080:8080
    volumes:
      - /path/to/config.yaml:/app/config.yaml
    networks:
      - chatgpt-ns
````



