---
title: MCP的知识
excerpt: "MCP的知识"
date: 2025-06-04 13:53:01
tag: [MCP]
categories: [MCP]
---

## conda安装 uv包管理器 供MCP使用

运行conda 激活环境

```cmd
conda activate myuv
```

使用pip在虚拟环境中安装uv包管理器

```cmd
pip install uv
```

安装对应的MCP服务,如：mysql_mcp_server

```cmd
pip install mysql_mcp_server
```

在mcp服务中的配置,其中command 填写conda环境下的uv的绝对路径

```json

{
  "mcpServers": {
    "mysql": {
      "command":"D:/anaconda/envs/myuv/Scripts/uv.exe",
      "args": [
        "run",
        "--directory",
        "D:/anaconda/envs/myuv/Lib/site-packages/mysql_mcp_server",
        "mysql_mcp_server"
      ],
      "env": {
        "MYSQL_DATABASE": "mcp",
        "MYSQL_HOST": "localhost",
        "MYSQL_PASSWORD": "root",
        "MYSQL_PORT": "3306",
        "MYSQL_USER": "root"
      }
    }
  }
}

```





