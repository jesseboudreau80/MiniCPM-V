<div align="center">

# OmniLMM
**Large multi-modal models for strong performance and efficient deployment**
<!-- <p align="center">
  <a href="#-viscpm-chat">Multimodal Conversation Model VisCPM-Chat</a> •
  <a href="#-viscpm-paint">Text-to-image Model VisCPM-Paint</a> •
  <a href="#-inference">Inference</a> •
  <a href="https://arxiv.org/pdf/2308.12038.pdf">Paper</a>
</p> -->

</div>


OmniLMM is a family of open-source large multimodal models (LMMs) that are adept at vision & language modeling. The model accepts images and text input, and emitts text outputs. We release two versions of OmniLMM that are targeted at strong performance and efficient deployment.
- OmniLMM 12B, the most capable version that achives state-of-the-art performance among open-source models with comparable sizes on MMMU.
- OmniLMM 3B, the efficient version that can be deployed on edge devices with promising performance.

## OmniLMM 12B
OmniLMM 12B is the most capable version. The model is built based on EVA-E 5B and Zephyr 7B, and trained on mulitmodal data in a curriculum fashion. It has three notable features:

- **Strong Performance.** OmniLMM 12B achieves state-of-the-art performance on MMMU among open-source models with comparable sizes, surpassing established open-source LMMs on mulitple benchmarks (including MME, MMBench and SEED-Bench, etc). The model also supports OCR capability and endows rich multimodal world knowledge.

- **Trustworthy Behavior.** LMMs are known for suffering from hallucination, often generating text that is not factually grounded in images (e.g., faithfully describing non-existing objects in images). OmniLMM 12B is first state-of-the-art open-source LMM aligned via multimodal RLHF for trustworthy behavior, and ranked #1 among open-source models on MMHalBench and Object Halbench.

- **Realtime Mulitmodal Interaction.** We combine the OmniLMM 12B and ChatGPT3.5 into a realtime multimodal interactive assistant. The assistant accepts video stream from camera and speech stream from microphone, and emitts speech output. While still primary, we find the model can replicate some of the fun cases shown in Gemini Demo video, without any video edition.

TODO：实验结果，可以放个表格截图（基准：MMMU、MME、MMBench、SEED-Bench、MMHALBench、Object Halbench；模型：Qwen-VL-Chat、CogVLM、GPT-4V、Gemini等） @余天予

TODO：case画图展示 @蔡天驰

## OmniLMM 3B
OmniLMM 3B (i.e., MiniCPM-Omni) is an efficient version for deployment. The model is built based on SigLip 400M and MiniCPM 2.4B, and trained in a smilar way to OmniLMM 12B. Notable features include:

- **High Efficiency.** OmniLLM 3B can be efficiently deployed on most GPU cards and personal computers, and even on edge devices such as mobilephones. Due to the significantly fewer tokens used to represent the images (i.e., 64 in OmniLMM 3B vs. 512+ in counterpart models), OmniLMM 3B can operate with less memory cost and higher speed during inference.

- **Promising Performance.** OmniLMM 3B achieves promising performance on multiple benchmarks (including MMMU, MME and MMbech), surpassing existing LMMs built on Phi-2. The model supports bilingual mulitmodal interaction in English and Chinese, and endows rich multimodal world knowledge.

TODO：实验结果，可以放个表格截图 @王崇屹

TODO：视频展示手机端效果？ @蔡天驰

<table>
  <tr>
    <td>
      <p> 
        <img src="data/Mushroom_en.gif" width="400"/>
      </p>
    </td>
    <td>
      <p> 
        <img src="data/Snake_en.gif" width="400"/>
      </p>
    </td>
  </tr>
  <tr>
    <td colspan="2" align="center">
      <p>🌐 Web Demo for both CogVLM and CogAgent: <a href="http://36.103.203.44:7861">this link</a></p>
    </td>
  </tr>
</table>


## Get Started

## ⚙️ Install

1. Clone this repository and navigate to source folder

```bash
git clone https://github.com/OpenBMB/OmniLMM.git
cd OmniLMM
```

2. Create conda environment

```Shell
conda create -n OmniLMM python=3.10 -y
conda activate OmniLMM
```

3. Install dependencies

```shell
pip install -r requirements.txt
```

## 💡 Inference

### Model Zoo
| Model                | Description       | Download Link |
|----------------------|-------------------|---------------|
| OmniLMM-12B | OmniLMM 12B is the most capable version                                 | [download](https://huggingface.co/openbmb/OmniLMM-12B/blob/main/pytorch_model.v1.bin) |
| OmniLMM-3B  | OmniLMM 3B (i.e., MiniCPM-Omni) is an efficient version for deployment. | [download](https://huggingface.co/openbmb/OmniLMM-3B/blob/main/pytorch_model.v1.bin)  |

### VisCPM-Chat
After downloading the checkpoints, please refer to the following codes to run `OmniLMM` (replace `'/path/to/checkpoint'` with actually path of downloaded checkpoint).

#### Single-turn Conversation

<div align="center">
<img src="data/COCO_test2015_000000262144.jpg" width="660px">
</div>

We can have a multimodal conversation with VisCPM-Chat using a few lines of codes.
```shell
# If the memory of your GPU is less than 40G, you can introduce the following environment variables. After the introduction, the memory usage is about 17G, but the time required for inference will be longer. This feature relies on the BMInf package.
export CUDA_MEMORY_CPMBEE_MAX=1g
```

```python
# Load and initialize the model
model_path = '/path/to/checkpoint'
chat_model = OmniLMM(model_path)

# We perform security checks on the input images by default.
im_64 = img2base64('./data/COCO_test2015_000000262144.jpg')

# First round chat 
msgs = [{"role": "user", "content": "What are the people doing?"}]
input = {
    "image": im_64,
    "question": json.dumps(msgs, ensure_ascii=True)
}
answer = chat_model.process(input)
print(answer)
```

We can obtain the following results:
```
"The people in the image are playing baseball. One person is pitching a ball, another one is swinging a bat to hit it, and there's also an umpire present who appears to be watching the game closely."
```

#### Multi-turn Conversation

```python
# Load and initialize the model
model_path = '/path/to/checkpoint'
chat_model = OmniLMM(model_path)

# We perform security checks on the input images by default.
im_64 = img2base64('./data/COCO_test2015_000000262144.jpg')

# First round chat 
msgs = [{"role": "user", "content": "What are the people doing?"}]
input = {
    "image": im_64,
    "question": json.dumps(msgs, ensure_ascii=True)
}
answer = chat_model.process(input)
print(answer)

# Second round chat 
# pass history context of multi-turn conversation
msgs.append({"role": "assistant", "content": answer})
msgs.append({"role": "user", "content": "Describe the image"})
input = {
    "image": im_64,
    "question": json.dumps(msgs, ensure_ascii=True)
}
answer = chat_model.process(input)
print(answer)
```

We can obtain the following results:
```
"The people in the image are playing baseball. One person is pitching a ball, another one is swinging a bat to hit it, and there's also an umpire present who appears to be watching the game closely."

"The image depicts a baseball game in progress. A pitcher is throwing the ball, while another player is swinging his bat to hit it. An umpire can be seen observing the play closely."
```

TODO：使用文档（安装、使用、提供Demo入口，包括3B和12B） @朱宏吉

## 🏫 Institutions

This project is developed by the following institutions:

- <img src="figures/thunlp.png" width="28px"> [THUNLP](https://nlp.csai.tsinghua.edu.cn/)
- <img src="figures/modelbest.png" width="28px"> [ModelBest](https://modelbest.cn/)
- <img src="figures/zhihu.webp" width="28px"> [Zhihu](https://www.zhihu.com/ )


