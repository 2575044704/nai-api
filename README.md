# nai-api
API for novelai.net
## 导演模式
```python
import requests
import base64
import os
from PIL import Image

def augment_image(api_key, image_path, output_path):
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

    with open(image_path, "rb") as image_file:
        image_base64 = base64.b64encode(image_file.read()).decode("utf-8")

    # 获取图片的原始尺寸
    with Image.open(image_path) as img:
        width, height = img.size
        print(f"图片尺寸：{width}x{height}")

    data = {
        "req_type": "declutter",
        "width": width,  # 使用图片的原始宽度
        "height": height,  # 使用图片的原始高度
        "image": image_base64
    }

    response = requests.post(url, headers=headers, json=data)

    # 打印调试信息
    print(f"响应： {response.status_code}")

    if response.status_code == 200:
        with open(output_path, "wb") as output_file:
            output_file.write(response.content)
        print(f"保存至 {output_path}")
        return output_path
    else:
        print("Error:", response.status_code, response.text)
        return None

# 示例调用
api_key = "pst-AumpmK4N7A0Isd2dG5SGPVnDG8lyKR***b"
image_path = "test.jpg"  # 上传的图片文件路径
output_dir = "naioutput"  # 确保目录存在
os.makedirs(output_dir, exist_ok=True)
output_path = os.path.join(output_dir, "processed_image.zip")


result = augment_image(api_key, image_path, output_path)

if result:
    print(f"图片保存至 {result}")

```
## **说明**
**req_type可选：**

> bg-removal=去背景，会返回3张图

> lineart=轮廓

> sketch=草图

> colorize=上色，给草图或者轮廓图上色
> Defry=0-5
> Prompt有提示词
> prompt: " eve (black cat), ", defry: 1


> emotion=换表情，可选Neutral，Happy，Sad，Angry，Scared，Surprised，Tired，Excited，Nervous，Thinking，Confused，Shy，Disgusted，Smug，Bored，Laughing，Irritated，Aroused，Embarrassed，Worried，Love，》》Determined，Hurt，Playful，
prompt: playful;;测试
> ;;是分割，前面是表情，后面是提示词

> declutter=删除图片气泡或文本

**生成花费**
python代码采用自适应图片分辨率，原图越大，消耗的点数越大

## 效果图（以declutter为例）
| 原图                                            | 输出                                            |
|-------------------------------------------------|-------------------------------------------------|
| ![119684319_p14](https://github.com/user-attachments/assets/eb045af7-ae80-4846-b54f-a57c34950f13) | ![image_0](https://github.com/user-attachments/assets/d80cf888-188c-4c65-813a-ea765891dd02) |

