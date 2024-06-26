# llama.cs for Unity
[llama.cs](https://assetstore.unity.com/packages/tools/ai-ml-integration/llama-cs-275927) is a simple implementation of an **LLM Chat** built on top of **llama.cs**, the C# binding for [llama.cpp](https://github.com/ggerganov/llama.cpp). It includes llama.cs, high-level APIs like LLM, LLMHost, and a Chat UI.

![llama.cs][llava]

Youtube: https://www.youtube.com/watch?v=6V4KhO6lM04

## Introduction
This asset serves as an excellent starting point for exploring and utilizing Large Language Models (LLMs) within the Unity environment. It comes equipped with the llama.cs binding, precompiled binaries for Windows (instructions for compiling binaries on other platforms, available in the [llama.cpp](https://github.com/ggerganov/llama.cpp) repository), and a straightforward and simplistic user interface. Whether you are a developer looking to integrate LLMs into your Unity projects or an enthusiast eager to experiment with language models, this asset provides the essential tools.

## Features
- Single-file llama.cs: The C# binding for llama.cpp
- High-level LLM wrapper implementation
- Optimized Chat UI with Virtualized Scroll View

<div style="page-break-after: always;"></div>

## System Requirements
Please refer to [this section](https://github.com/ggerganov/llama.cpp?tab=readme-ov-file#memorydisk-requirements) for information on memory and disk requirements. 
The recommended machine is one with an 8GB Nvidia GPU, a modern processor, and 32GB RAM.

## Getting Started
1. Open Assets/Battlehub/Chat scene
2. Download **Meta-Llama-3-8B-Instruct.Q4_K_M.gguf** [here](https://huggingface.co/QuantFactory/Meta-Llama-3-8B-Instruct-GGUF/tree/main). 
3. Copy Meta-Llama-3-8B-Instruct.Q4_K_M.gguf to **Assets/StreamingAssets** folder 
4. Enter play mode

> **Note**
You can use the following link to download the model:
[https://huggingface.co/QuantFactory/Meta-Llama-3-8B-Instruct-GGUF/resolve/main/Meta-Llama-3-8B-Instruct.Q4_K_M.gguf?download=true](https://huggingface.co/QuantFactory/Meta-Llama-3-8B-Instruct-GGUF/resolve/main/Meta-Llama-3-8B-Instruct.Q4_K_M.gguf?download=true)


> **Note**
You might want to replace llama.dll and llava_shared.dll with one that fits your platform (see **Assets\Battlehub\LLama\Plugins** folder):
[https://github.com/ggerganov/llama.cpp/releases/tag/b3212](https://github.com/ggerganov/llama.cpp/releases/tag/b3212)

> **Note**
if you have an NVIDIA GPU, use the cuda version for best performance:

> **Note**
The current supported version is b3212. 

> **Note**
To build llama.cpp from source, please refer to the following section [https://github.com/ggerganov/llama.cpp?tab=readme-ov-file#usage](https://github.com/ggerganov/llama.cpp?tab=readme-ov-file#usage)

![Getting Started Result][getting_started_result]

## Definitions
### llama.cs
A single-file P/Invoke binding for llama.cpp.

### LLM.cs
High-level wrapper for the llama.cs API.

### LLMHost.cs 
MonoBehaviour responsible for loading/unloading LLM and dispatching calls from the main to LLM thread.

### Chat UI
Implementation of Chat UI, comprising ChatUI.prefab, ChatUI.cs, and VirtualScroll.cs.

<div style="page-break-after: always;"></div>

## Config Files
Config files are JSON files with serialized **gpt_params** structure. Example config files can be found in the **Assets/StreamingAssets/Configs** folder. empty-gpt_params.json is an empty template with all available parameters, while llama-3-8B-instruct-gpt_params is a config file used for demonstration purposes in the Getting Started section. On Mac use llama-3-8B-instruct-osx_gpt_params

To load a config file using the Chat UI, use the "Load gpt params" button.

![Load GPT Params button][load_gpt_params]

## Examples
### Dummy LLM Implementation

This example demonstrates how to replace the default LLM implementation with your own.

```C#
using System.Collections.Generic;
using System.Threading;

namespace Battlehub.LLama.Examples
{
    public class DummyLLM : ILLM
    {
        private string m_input;

        public IEnumerator<Response> Initialize()
        {
            yield return new Response("Initializing", ResponseType.Log);
            yield return new Response("Initialized", ResponseType.InitCompleted);
        }

        public void Input(string input, byte[][] images = null)
        {
            m_input = input;
        }

        public IEnumerator<Response> Loop()
        {
            yield return new Response("Chat started. Enter \"Stop\" to end.", ResponseType.Log);
            while (true)
            {
                yield return new Response(ResponseType.Eos);
                
                if (m_input == "Stop")
                {
                    break;
                }

                yield return new Response(ResponseType.Bos);

                string[] tokens = m_input.Split(' ');
                for (int i = 0; i < tokens.Length; i++) 
                {
                    Thread.Sleep(100);
                    yield return new Response($"{tokens[i]} ", ResponseType.Token);
                }
            }

            yield return new Response("Chat Ended", ResponseType.Log);
            yield return new Response(ResponseType.EndOfText);
        }

        public void Dispose()
        {
        }
    }
}

namespace Battlehub.LLama.Examples
{
    public class DummyLLMHost : LLMHost
    {
        protected override ILLM CreateLLM()
        {
            return new DummyLLM();
        }
    }
}
```

> **Note**  
The complete example can be found in Assets/Battlehub/LLama/Examples/DummyLLM.

![Dummy LLM][dummy_llm]

### LLM Client

This example illustrates how to utilize the LLM host for communication with LLM.

```C#
using System.IO;
using System.Text;
using System.Threading.Tasks;
using UnityEngine;

namespace Battlehub.LLama.Examples
{
    public class LLMClient : MonoBehaviour
    {
        private ILLMHost m_host;

        [SerializeField]
        private string m_configPath;

        private void Start()
        {
            if (!File.Exists($"{Application.streamingAssetsPath}/Meta-Llama-3-8B-Instruct.Q4_K_M.gguf"))
            {
                Debug.LogWarning("Download Meta-Llama-3-8B-Instruct.Q4_K_M.gguf and move it to the StreamingAssets folder. <a href=\"https://huggingface.co/QuantFactory/Meta-Llama-3-8B-Instruct-GGUF/resolve/main/Meta-Llama-3-8B-Instruct.Q4_K_M.gguf?download=true\">https://huggingface.co/QuantFactory/Meta-Llama-3-8B-Instruct-GGUF/resolve/main/Meta-Llama-3-8B-Instruct.Q4_K_M.gguf?download=true</a>");
            }

            if (string.IsNullOrEmpty(m_configPath))
            {
                m_configPath = $"{Application.streamingAssetsPath}/configs/llama-3-8B-instruct-gpt_params.json";
            }

            m_host = gameObject.AddComponent<LLMHost>();
            m_host.Response += OnResponse;
            m_host.ConfigPath = m_configPath;
        }

        private StringBuilder m_stringBuilder = new StringBuilder();
        private async void OnResponse(Response response)
        {
            if (response.ResponseType == ResponseType.InitCompleted)
            {
                Debug.Log("InitCompleted");
            }
            else if (response.ResponseType == ResponseType.Bos)
            {
                m_stringBuilder.Clear();
            }
            else if (response.ResponseType == ResponseType.Token)
            {
                m_stringBuilder.Append(response.Data);
            }
            else if (response.ResponseType == ResponseType.Eos)
            {
                await Task.Yield();

                if (m_stringBuilder.Length == 0)
                {
                    const string prompt1 = "Hi, Friend. Come up with a few lines of a story";
                    Debug.Log(prompt1);

                    m_host.SendRequest(prompt1);
                }
                else
                {
                    Debug.Log(m_stringBuilder.ToString());
                    m_stringBuilder.Clear();

                    const string prompt2 = "What happened next?";
                    Debug.Log(prompt2);

                    m_host.SendRequest(prompt2);
                }
            }
            else
            {
                Debug.Log(response.Data);
            }
        }
    }
}

```

> **Note**  
The complete example can be found in Assets/Battlehub/LLama/Examples/LLMClient.

![LLM Client][llm_client]

## Support
If you cannot find something in the documentation or have any questions, please feel free to send an email to [Battlehub@outlook.com](mailto:Battlehub@outlook.com) or ask directly in [this](https://t.me/battlehub) support group. Keep up the great work in your development journey! 😊

[llava]:  Docs/Images/llava.png
[getting_started_result]:  Docs/Images/getting_started.png
[dummy_llm]:  Docs/Images/dummy_llm.png
[llm_client]:  Docs/Images/llm_client.png
[load_gpt_params]: Docs/Images/load_gpt_params.png



