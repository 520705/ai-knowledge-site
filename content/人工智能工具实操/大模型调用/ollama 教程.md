## 是啥

Ollama 是一款轻量级、可扩展的框架，旨在帮助用户在本地计算机上便捷地部署和运行大型语言模型（LLM）。 它支持多种模型架构，包括 Llama 2、Gemma、Code Llama、Mistral 等， 并提供与 OpenAI 兼容的 API 接口，适合开发者和企业快速搭建本地化的大模型应用。 ​[arxiv.org+9blog.csdn.net+9zhuanlan.zhihu.com+9](https://blog.csdn.net/Everly_/article/details/140767440)[zhuanlan.zhihu.com+1blog.csdn.net+1](https://zhuanlan.zhihu.com/p/20642808493)

## 能干啥用

- **本地运行大型语言模型**：​Ollama 允许用户在本地设备上运行各种大型语言模型，如 Llama 2、Code Llama 等，满足对数据隐私和安全性的需求。 ​
    
- **模型管理**：​用户可以轻松下载、安装和管理多个模型，方便地在不同模型之间切换或进行比较。 ​[zhuanlan.zhihu.com+1zhuanlan.zhihu.com+1](https://zhuanlan.zhihu.com/p/692343935)
    
- **API 服务**：​Ollama 提供 RESTful API 服务，使开发者能够将本地模型集成到其他应用程序或服务中，扩展应用场景。 ​[blog.csdn.net+1zhuanlan.zhihu.com+1](https://blog.csdn.net/Everly_/article/details/140767440)
    

## 咋用

1. **安装 Ollama**：
    
    - **macOS 和 Windows**：​从官方网站下载对应的安装包并安装。 ​
        
    - **Linux**：​使用以下命令安装：​
        
        bash
        
        `curl -fsSL https://ollama.com/install.sh | sh`
        
        安装完成后，检查服务状态：
        
        bash
        
        `systemctl status ollama`
        
        确认安装成功：
        
2. **下载并运行模型**：
    
    - 使用 `ollama pull <模型名称>` 下载所需模型，例如：​
        
    - 运行模型：​
        
        此时，可以在终端中与模型进行交互。
        
3. **提供 API 服务**：
    
    - 启动 Ollama 的 API 服务：​[zhuanlan.zhihu.com+6blog.csdn.net+6zhuanlan.zhihu.com+6](https://blog.csdn.net/Everly_/article/details/140767440)
        
    - 通过发送 HTTP 请求，与模型进行交互。例如，生成回复：​
        
        bash
        
        `curl http://localhost:11434/generate -d '{"model": "llama2", "prompt": "介绍一下Ollama的功能。"}'`
        

## 参考资料

- [本地部署大模型？Ollama 部署和实战，看这篇就够了 - 知乎专栏](https://zhuanlan.zhihu.com/p/710560829)
    
- [Ollama完全指南：从入门到精通 - 知乎专栏](https://zhuanlan.zhihu.com/p/20748560635)
    
- [基于Ollama内部署本地大语言模型 - 知乎专栏](https://zhuanlan.zhihu.com/p/683950216)
    
- [Ollama 使用指南 - YouTube 视频](https://www.youtube.com/watch?v=POf4qbohP9k)
    

通过上述步骤，您可以在本地环境中部署并运行大型语言模型，满足个性化应用需求。​[zhuanlan.zhihu.com+5blog.csdn.net+5zhuanlan.zhihu.com+5](https://blog.csdn.net/Everly_/article/details/140767440)
- Ollama 部署本地大模型与使用 - 子洋的文章 - 知乎
https://zhuanlan.zhihu.com/p/17482899149


