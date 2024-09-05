# nai-api
API for novelai.net
## 首先要获取NovelAI API key：
### 1.首先进入[Novelai官网](https://novelai.net/stories)，点击左上角中间那个齿轮标志
![image](https://github.com/user-attachments/assets/7593250e-977b-4c29-b9f2-5c2a2211edd0)
### 2.左侧栏找到Account，点击Get Persistent API Token：
![image](https://github.com/user-attachments/assets/616d75e0-e769-4214-ab09-7e723997257c)
### 3.将获得的API token复制下来
![image](https://github.com/user-attachments/assets/84e4670c-f682-48fe-a190-7124da80de71)

## NovelAi Director图像工具API调用：
```python
import requests
import base64
import os
import zipfile
from PIL import Image

#↓把你复制的api key填写在引号内
api_key = ""

#上传数据
req_type = "declutter"  #见底部说明
prompt = "angry;; solo, "  # 提示词（只有上色、换表情需要。不是所有请求类型都需要）
defry = 1 （只有上色需要，0~5）

# 上传的图片文件路径和输出目录
image_path = "test.jpg"     # 上传的图片文件路径
output_dir = "naioutput"    # 输出目录

def augment_image(image_path, output_dir, req_type, prompt, defry):
    url = "https://image.novelai.net/ai/augment-image"
    headers = {
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json"
    }

    # 确保路径没有嵌入的 null 字符
    image_path = image_path.strip()

    if not os.path.exists(image_path):
        print(f"Error: 文件 {image_path} 不存在.")
        return None

    # 获取图片的原始尺寸
    with Image.open(image_path) as img:
        width, height = img.size
        print(f"图片尺寸：{width}x{height}")

    # 读取图片并编码为 base64
    with open(image_path, "rb") as image_file:
        image_base64 = base64.b64encode(image_file.read()).decode("utf-8")

    # 发送数据，使用外部传入的 req_type, prompt 和 defry
    data = {
        "req_type": req_type,
        "prompt": prompt,
        "defry": defry,
        "width": width,        # 使用图片的原始宽度
        "height": height,      # 使用图片的原始高度
        "image": image_base64
    }

    # 发送请求
    response = requests.post(url, headers=headers, json=data)

    # 打印调试信息
    print(f"响应： {response.status_code}")

    if response.status_code == 200:
        # 定义 ZIP 文件输出路径，不包含多余子目录
        zip_path = os.path.join(output_dir, "processed_image.zip")
        
        # 确保目标目录存在
        os.makedirs(output_dir, exist_ok=True)
        
        # 将响应内容写入 ZIP 文件
        with open(zip_path, "wb") as output_file:
            output_file.write(response.content)
        print(f"ZIP 文件保存至 {zip_path}")

        # 自动解压缩处理后的图片到输出目录
        with zipfile.ZipFile(zip_path, 'r') as zip_ref:
            for member in zip_ref.namelist():
                filename = os.path.basename(member)
                if filename:
                    file_path = os.path.join(output_dir, filename)
                    with open(file_path, 'wb') as f:
                        f.write(zip_ref.read(member))
                    print(f"已提取文件：{file_path}")

        # 提取图片文件路径
        extracted_images = [os.path.join(output_dir, f) for f in os.listdir(output_dir) if f.endswith(('.png', '.jpg', '.jpeg'))]
        if extracted_images:
            print(f"提取到的图片文件：{extracted_images}")
        else:
            print("未提取到图片文件.")
        
        return extracted_images
    else:
        print("Error:", response.status_code, response.text)
        return None


os.makedirs(output_dir, exist_ok=True)


result = augment_image(image_path, output_dir, req_type, prompt, defry)

if result:
    print(f"图片保存至 {result}")

```
## **说明**
**req_type可选：**

> bg-removal=去背景，会返回3张图

> lineart=轮廓

> sketch=草图

> colorize=上色，给草图或者轮廓图上色
> 可选Defry=0-5
> 可选Prompt有提示词
> 例：prompt: " eve (black cat), ", defry: 1


> emotion=换表情，可选Neutral，Happy，Sad，Angry，Scared，Surprised，Tired，Excited，Nervous，Thinking，Confused，Shy，Disgusted，Smug，Bored，Laughing，Irritated，Aroused，Embarrassed，Worried，Love，Determined，Hurt，Playful，
>
> prompt: playful;;
> ;;是分割，前面是表情，后面是提示词

> declutter=删除图片气泡或文本

**生成花费**：
python代码采用自适应图片分辨率，原图越大，消耗的点数越多

## 效果图（以declutter为例）
| 原图                                            | 输出                                            |
|-------------------------------------------------|-------------------------------------------------|
| ![119684319_p14](https://github.com/user-attachments/assets/eb045af7-ae80-4846-b54f-a57c34950f13) | ![image_0](https://github.com/user-attachments/assets/d80cf888-188c-4c65-813a-ea765891dd02) |

