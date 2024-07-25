# QWEN-Audio安装文档

## Step 1 

进入https://hf-mirror.com/Qwen/Qwen-Audio-Chat，点击Files and versions，下载所有文件，打包成一个文件夹，命名Qwen-Audio-Chat。

## Step 2 

在dockerhub拉取pythorch镜像(选择devel版本)。例如：

![image-20240725155344645](C:\Users\jdsxj\AppData\Roaming\Typora\typora-user-images\image-20240725155344645.png)

pytorch和cuda的版本参考Qwen-Audio-Chat要求。

![image-20240725155406141](C:\Users\jdsxj\AppData\Roaming\Typora\typora-user-images\image-20240725155406141.png)

## Step 3 

打开pytorch镜像，看是否需要升级镜像自带的python版本，并安装requirements.txt里的依赖包。（如果出现某个包找不到，可以试试清华源， pip install XX -i https://pypi.tuna.tsinghua.edu.cn/simple）

![image-20240725155748438](C:\Users\jdsxj\AppData\Roaming\Typora\typora-user-images\image-20240725155748438.png)

![image-20240725155810684](C:\Users\jdsxj\AppData\Roaming\Typora\typora-user-images\image-20240725155810684.png)

## Step 4 

以下是跑通的代码，来源于https://hf-mirror.com/Qwen/Qwen-Audio-Chat

注意几个修改的地方：

将第6行的from_pretrained()中的第一个参数，改成本地Qwen-Audio-Chat文件夹的路径

第19行'audio'之后，是本地音频文件的路径。

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from transformers.generation import GenerationConfig
import torch
torch.manual_seed(1234)
# Note: The default behavior now has injection attack prevention off.
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen-Audio-Chat", trust_remote_code=True)
# use bf16
# model = AutoModelForCausalLM.from_pretrained("Qwen/Qwen-Audio-Chat", device_map="auto", trust_remote_code=True, bf16=True).eval()
# use fp16
# model = AutoModelForCausalLM.from_pretrained("Qwen/Qwen-Audio-Chat", device_map="auto", trust_remote_code=True, fp16=True).eval()
# use cpu only
# model = AutoModelForCausalLM.from_pretrained("Qwen/Qwen-Audio-Chat", device_map="cpu", trust_remote_code=True).eval()
# use cuda device
model = AutoModelForCausalLM.from_pretrained("Qwen/Qwen-Audio-Chat", device_map="cuda", trust_remote_code=True).eval()
# Specify hyperparameters for generation (No need to do this if you are using transformers>4.32.0)
# model.generation_config = GenerationConfig.from_pretrained("Qwen/Qwen-Audio-Chat", trust_remote_code=True)
# 1st dialogue turn
query = tokenizer.from_list_format([
    {'audio': 'https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen-Audio/1272-128104-0000.flac'}, # Either a local path or an url
    {'text': 'what does the person say?'},
])
response, history = model.chat(tokenizer, query=query, history=None)
print(response)
# The person says: "mister quilter is the apostle of the middle classes and we are glad to welcome his gospel".

# 2nd dialogue turn
response, history = model.chat(tokenizer, 'Find the start time and end time of the word "middle classes"', history=history)
print(response)

```

