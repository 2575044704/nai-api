# nai-api
API for novelai.net
## 导演模式
```python
import requests
import base64
import os

def augment_image(api_key, image_path, output_path):
    url = "https://image.novelai.net/ai/augment-image"
    headers = {
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json"
    }

    # 确保路径没有嵌入的 null 字符
    image_path = image_path.strip()

    if not os.path.exists(image_path):
        print(f"Error: The file {image_path} does not exist.")
        return None

    with open(image_path, "rb") as image_file:
        image_base64 = base64.b64encode(image_file.read()).decode("utf-8")

    data = {
        "req_type": "sketch",
        "width": 832,
        "height": 1216,
        "image": image_base64
    }

    response = requests.post(url, headers=headers, json=data)
    
    # 打印调试信息
    print(f"Status Code: {response.status_code}")

    if response.status_code == 200:
        with open(output_path, "wb") as output_file:
            output_file.write(response.content)
        print(f"Processed image saved to {output_path}")
        return output_path
    else:
        print("Error:", response.status_code, response.text)
        return None

# 示例调用
api_key = "pst-N0j6NNTF0IxJW*************************"
image_path = "image_0.png"  # 上传的图片文件路径
output_dir = "output"  # 确保目录存在
os.makedirs(output_dir, exist_ok=True)
output_path = os.path.join(output_dir, "processed_image.zip")


result = augment_image(api_key, image_path, output_path)

if result:
    print(f"Processed image is saved in: {result}")
```
**说明**
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
