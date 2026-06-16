# LocalLLM_Benchmarking
# Local LLM Hardware Telemetry & Benchmarking Engine
An automated, Python-based telemetry pipeline utilizing `psutil` to monitor OS-level system vitals (Page Faults, RAM Deltas, CPU Saturation, and Disk I/O) during local AI inference. The objective of this project was to establish a Trade-Off Matrix for deploying Quantized LLMs on constrained edge-hardware, identifying exact hardware bottlenecks during variable context-window scaling.

Models were executed sequentially via the Ollama engine. System cache was flushed prior to execution. Each model was subjected to a 3-Tier prompt escalation:
Tier 1 (Short Burst):** Tests TTFT (Time to First Token) and disk-to-RAM load latency.
Tier 2 (Logic Loop):** Tests basic reasoning token-throughput.
Tier 3 (Context Flooder):** Tests KV-Cache memory degradation and CPU saturation during sustained generation.

### The Trade-Off Matrix (Raw Telemetry Data)

| Model | Prompt Tier | RAM Delta (MB) | Page Faults | TTFT (Sec) | Tokens/Sec | CPU Saturation | Disk Read (MB) |
|---|---|---|---|---|---|---|---|
| **gemma2:2b** | T1: Burst | 2074.48 | 172 | 7.04 | 0.54 | 52.02% | 1617.55 |
| **gemma2:2b** | T2: Logic | 273.61 | 9 | 2.30 | 8.11 | 72.62% | 272.84 |
| **gemma2:2b** | T3: Context | 87.58 | 9 | 2.00 | 7.78 | 76.50% | 632.86 |
| **phi3:latest** | T1: Burst | 3561.97 | 6 | 11.23 | 0.26 | 65.23% | 2159.12 |
| **phi3:latest** | T2: Logic | 37.27 | 2 | 1.52 | 7.13 | 73.13% | 48.12 |
| **phi3:latest** | T3: Context | 767.62 | **212** | 3.00 | 4.39 | **96.34%** | 3337.27 |
| **llama3:latest**| T1: Burst | **4039.95** | 17 | **21.68** | 0.09 | 44.35% | **4470.23** |
| **llama3:latest**| T2: Logic | 43.85 | 1 | 4.38 | 3.21 | 63.75% | 53.96 |
| **llama3:latest**| T3: Context | 46.81 | 17 | 5.78 | 3.42 | 60.70% | 401.47 |

### Engineering Conclusion
For this specific hardware environment (16GB Total RAM), **Gemma 2 (2B)** is the only production-viable model. While Llama 3 (8B) successfully executed instructions without crashing the OS, its 4.4GB memory footprint caused a severe 21-second cold-load latency, rendering it unsuitable for real-time application deployment. Phi 3 experienced severe computational bottlenecking on long-context generation (96% CPU saturation).
