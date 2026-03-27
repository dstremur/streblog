---
title: "Benchmarking the Instinct MI50 in 2026"
date: 2026-03-15T20:47:22+01:00
tags: ["GPU", "AMD"]
categories: ["Review"]
---
## Introduction 
This blog post serves as a collection of benchmarks regarding the Instinct MI50. 
It will probably be extended in the future, see the changelog at the bottom.

## LLM benchmarks

First, I ran some LLM benchmarks using `llama-bench`. All models were downloaded from [Huggingface](https://huggingface.co/)
in a GGUF format. The default number of test rounds was increased to 50 and Flash-Attention was enabled. 
### Apertus-70B-Instruct

Kicking things off, we have the Swiss [Apertus-70B-Instruct][1] Model. The MI50 handled this 41GB model very well, 
resulting in ~11 Token/s in generation. Sadly you can feel that the model is still way behind foreign options in terms of usability.

```
llama-bench -m ~/models/Apertus-70B-Instruct-2509-UD-Q4_K_XL.gguf -ngl 99 -fa 1 -r 50
ggml_cuda_init: found 4 ROCm devices (Total VRAM: 65472 MiB):
  Device 0: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 1: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 2: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 3: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
| model                          |       size |     params | backend    | ngl | fa |            test |                  t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | -: | --------------: | -------------------: |
| apertus ?B Q4_K - Medium       |  41.17 GiB |    70.60 B | ROCm       |  99 |  1 |           pp512 |         92.93 ± 0.01 |
| apertus ?B Q4_K - Medium       |  41.17 GiB |    70.60 B | ROCm       |  99 |  1 |           tg128 |         11.34 ± 0.02 |

build: 43e1cbd (8255)
```
### Qwen3.5-27B

Next is the ever popular [Qwen3.5-27B][2] first in a 4-bit quantization. 

```
llama-bench -m ~/models/Qwen3.5-27B.Q4_K_M.gguf -ngl 99 -fa 1 -r 50
ggml_cuda_init: found 4 ROCm devices (Total VRAM: 65472 MiB):
  Device 0: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 1: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 2: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 3: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
| model                          |       size |     params | backend    | ngl | fa |            test |                  t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | -: | --------------: | -------------------: |
| qwen35 27B Q4_K - Medium       |  15.39 GiB |    26.90 B | ROCm       |  99 |  1 |           pp512 |        235.18 ± 0.05 |
| qwen35 27B Q4_K - Medium       |  15.39 GiB |    26.90 B | ROCm       |  99 |  1 |           tg128 |         16.11 ± 0.04 |

build: 43e1cbd (8255)
```
As a second test, I also ran the same [Qwen3.5-27B][3] in a 8-bit quantization.

```
llama-bench -m ~/models/Qwen3.5-27B.Q8_0.gguf -ngl 99 -fa 1 -r 50
ggml_cuda_init: found 4 ROCm devices (Total VRAM: 65472 MiB):
  Device 0: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 1: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 2: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 3: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
| model                          |       size |     params | backend    | ngl | fa |            test |                  t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | -: | --------------: | -------------------: |
| qwen35 27B Q8_0                |  26.62 GiB |    26.90 B | ROCm       |  99 |  1 |           pp512 |        134.94 ± 0.15 |
| qwen35 27B Q8_0                |  26.62 GiB |    26.90 B | ROCm       |  99 |  1 |           tg128 |         14.63 ± 0.06 |

build: 43e1cbd (8255)
```

### Qwen3.5-35B-A3B 

The [Qwen3.5-35B-A3B][4] seemed to perform very well on the hardware.

```
llama-bench -m ~/models/Qwen3.5-35B-A3B-Q8_0.gguf -ngl 99 -fa 1 -r 50
ggml_cuda_init: found 4 ROCm devices (Total VRAM: 65472 MiB):
  Device 0: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 1: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 2: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 3: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
| model                          |       size |     params | backend    | ngl | fa |            test |                  t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | -: | --------------: | -------------------: |
| qwen35moe 35B.A3B Q8_0         |  34.36 GiB |    34.66 B | ROCm       |  99 |  1 |           pp512 |        626.32 ± 3.88 |
| qwen35moe 35B.A3B Q8_0         |  34.36 GiB |    34.66 B | ROCm       |  99 |  1 |           tg128 |         35.48 ± 0.20 |

build: 43e1cbd (8255)
```

### Qwen3-Coder-Next 

For a coding focused model, I also tested [Qwen3-Coder-Next][5], which is the biggest model in this lineup at 46 GiB.

```
llama-bench -m ~/models/Qwen3-Coder-Next-UD-Q4_K_XL.gguf -ngl 99 -fa 1 -r 50
ggml_cuda_init: found 4 ROCm devices (Total VRAM: 65472 MiB):
  Device 0: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 1: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 2: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 3: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
| model                          |       size |     params | backend    | ngl | fa |            test |                  t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | -: | --------------: | -------------------: |
| qwen3next 80B.A3B Q4_K - Medium |  46.20 GiB |    79.67 B | ROCm       |  99 |  1 |           pp512 |        510.10 ± 2.71 |
| qwen3next 80B.A3B Q4_K - Medium |  46.20 GiB |    79.67 B | ROCm       |  99 |  1 |           tg128 |         28.58 ± 0.13 |

build: 43e1cbd (8255)
```

### Llama-3.3-70B-Instruct 

The [Llama-3.3-70B-Instruct][6] was also handled very well.

```
llama-bench -m ~/models/Llama-3.3-70B-Instruct-UD-Q4_K_XL.gguf -ngl 99 -fa 1 -r 50
ggml_cuda_init: found 4 ROCm devices (Total VRAM: 65472 MiB):
  Device 0: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 1: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 2: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 3: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
| model                          |       size |     params | backend    | ngl | fa |            test |                  t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | -: | --------------: | -------------------: |
| llama 70B Q4_K - Medium        |  39.73 GiB |    70.55 B | ROCm       |  99 |  1 |           pp512 |        101.13 ± 0.02 |
| llama 70B Q4_K - Medium        |  39.73 GiB |    70.55 B | ROCm       |  99 |  1 |           tg128 |         11.87 ± 0.02 |

build: 43e1cbd (8255)
```

### gpt-oss-20b 

Also the smaller [gpt-oss-20b][7] ran very well in a 16-bit quantization.

```
llama-bench -m models/gpt-oss-20b-F16.gguf -fa 1 -r 50
ggml_cuda_init: found 4 ROCm devices (Total VRAM: 65472 MiB):
  Device 0: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 1: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 2: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 3: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
| model                          |       size |     params | backend    | ngl | fa |            test |                  t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | -: | --------------: | -------------------: |
| gpt-oss 20B F16                |  12.83 GiB |    20.91 B | ROCm       |  99 |  1 |           pp512 |       1205.57 ± 8.94 |
| gpt-oss 20B F16                |  12.83 GiB |    20.91 B | ROCm       |  99 |  1 |           tg128 |         86.74 ± 0.61 |

build: 43e1cbd (8255)
```

## Server Configuration

All tests were performed on a Gigabyte G292-Z20 with an Epyc 7402P and 4 Radeon Instinct MI50. The server runs NixOS 25.11
and ROCm version 6.4.3. Below are system info outputs.


```
rocm-smi

=========================================== ROCm System Management Interface ===========================================
===================================================== Concise Info =====================================================
Device  Node  IDs              Temp    Power     Partitions          SCLK    MCLK    Fan     Perf  PwrCap  VRAM%  GPU%  
              (DID,     GUID)  (Edge)  (Socket)  (Mem, Compute, ID)                                                     
========================================================================================================================
0       1     0x66a1,   29182  36.0°C  22.0W     N/A, N/A, 0         930Mhz  350Mhz  14.51%  auto  250.0W  0%     0%    
1       2     0x66a1,   57422  32.0°C  17.0W     N/A, N/A, 0         930Mhz  350Mhz  14.51%  auto  250.0W  0%     0%    
2       3     0x66a1,   54426  35.0°C  19.0W     N/A, N/A, 0         930Mhz  350Mhz  14.51%  auto  250.0W  0%     0%    
3       4     0x66a1,   45423  33.0°C  15.0W     N/A, N/A, 0         930Mhz  350Mhz  14.51%  auto  250.0W  0%     0%    
========================================================================================================================
================================================= End of ROCm SMI Log ==================================================
```

```
rocm-smi -V
ROCM-SMI version: 3.1.0+unknown
ROCM-SMI-LIB version: 7.7.0
```

```
llama --version
ggml_cuda_init: found 4 ROCm devices (Total VRAM: 65472 MiB):
  Device 0: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 1: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 2: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
  Device 3: AMD Instinct MI60 / MI50, gfx906:sramecc+:xnack- (0x906), VMM: no, Wave Size: 64, VRAM: 16368 MiB (16348 MiB free)
version: 8255 (43e1cbd)
built with GNU 15.2.0 for Linux x86_64
```

## Thermals & Power Consumption

During all the tests the thermals were never an issue. The hottest GPU barely reached 70C, which is also due to 
the fact that only one of them does any real work. Therefore, the power consumption was also modest with around 700W at peak.
I expect that this number will go up if I run any compute workloads on the cards in the future.

## Changelog

March 2026: First LLM benchmarks. 





[1]: https://huggingface.co/unsloth/Apertus-70B-Instruct-2509-GGUF?show_file_info=Apertus-70B-Instruct-2509-UD-Q4_K_XL.gguf

[2]: https://huggingface.co/unsloth/Qwen3.5-27B-GGUF?show_file_info=Qwen3.5-27B-Q4_K_M.gguf

[3]: https://huggingface.co/unsloth/Qwen3.5-27B-GGUF?show_file_info=Qwen3.5-27B-Q8_0.gguf

[4]: https://huggingface.co/unsloth/Qwen3.5-35B-A3B-GGUF?show_file_info=Qwen3.5-35B-A3B-Q8_0.B-GGUF

[5]: https://huggingface.co/unsloth/Qwen3-Coder-Next-GGUF?show_file_info=Qwen3-Coder-Next-UD-Q4_K_XL.gguf

[6]: https://huggingface.co/unsloth/Llama-3.3-70B-Instruct-GGUF?show_file_info=Llama-3.3-70B-Instruct-UD-Q4_K_XL.gguf

[7]: https://huggingface.co/unsloth/gpt-oss-20b-GGUF?show_file_info=gpt-oss-20b-F16.gguf
