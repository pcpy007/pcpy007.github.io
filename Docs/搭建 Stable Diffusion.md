# 1 安装相关依赖包

使用 Homebrew 安装

```shell
brew install cmake protobuf rust python@3.10 git wget
```

# 2 克隆 webUI 版本库

```shell
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui
```

# 3 下载 Stable Diffusion models/checkpoint

下载 Model 常用网站：

1）C 站：

[Civitai | Stable Diffusion models, embeddings, LoRAs and more](https://civitai.com/)

2）官方

[Models - Hugging Face](https://huggingface.co/models?pipeline_tag=text-to-image&sort=downloads)

如比较流行的 Model：

https://huggingface.co/CompVis/stable-diffusion-v-1-4-original/resolve/main/sd-v1-4.ckpt



下载的模型放在克隆的版本库路径，./stable-diffusion-webui/models/Stable-diffusion/ 文件夹下

# 4 启动 webUI

执行启动：

```shell
cd ./stable-diffusion-webui
./webui.sh
```



# 5 报错及解决

直接执行上述命令报错：

Something went wrong
Expecting value: line 1 column 1 (char 0)

上网查了下说是开了代理的原因，关掉后好像还不行。

第二种方法，启动时带以下参数

```sh
./webui.sh --autolaunch --no-gradio-queue         
# autolaunch 自动打开浏览器页面

./webui.sh --medvram --autolaunch --deepdanbooru --xformers --no-gradio-queue
```

打开 127.0.0.1:7861，输入咒语后点击生成，执行成功。

![image-20230709123941054](https://github.com/pcpy007/pcpy007.github.io/blob/master/img/image-20230709123941054.png)

# 6 ref

[Installation on Apple Silicon · AUTOMATIC1111/stable-diffusion-webui Wiki (github.com)](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Installation-on-Apple-Silicon)