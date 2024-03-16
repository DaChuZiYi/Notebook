---
title: 文章编辑器
data: 2024-03-10 09:03:05 +0800
image_path: "/assets/img/"
categories: [Python, 程序]
tags: [Python , 文章 , 编辑 , 程序]
---

写了一个适用于Jekyll的文章编辑器

**特性**:

- 读取/生成Jekyll格式的文章
- 自动填充时间
- 自动格式化文件名
- 可视化头文件填写
- 自动创建在`_post`中
- 分类数量限制提示
- 必要信息填充提示

**注意事项**

- 请将程序放在`_post`同级目录下(如果有)

具体代码如下:

```python
# 模块安装检测
import importlib.util
import subprocess
import sys


def install_module(module_name):
    subprocess.check_call([sys.executable, "-m", "pip", "install", module_name])


def check_and_install(module_name):
    spec = importlib.util.find_spec(module_name)
    if spec is None:
        print(f"模块 {module_name} 好像还没装,正在帮你装上,等一会昂...")
        install_module(module_name)
        print(f"模块 {module_name} 装好啦~")


# 需要检查的模块列表
required_modules = ["tkinter", "configparser", "ttkbootstrap", "pyyaml"]

for module in required_modules:
    check_and_install(module)

# 模块导入
import tkinter as tk
from tkinter import messagebox
import ttkbootstrap as ttk
from ttkbootstrap.constants import *
import os
import configparser as con
from datetime import datetime
from tkinter import filedialog
import yaml

# 确定配置文件路径
# 获取当前文件所在的目录
current_directory = os.path.dirname(os.path.abspath(__file__))
# 构建配置文件的路径
config_path = os.path.join(current_directory, "pe.ini")
if not os.path.exists(config_path):
    with open(config_path, "w") as f:
        pass
# 设置
config = con.ConfigParser()
config.read(config_path)
if not config.has_section("PEsettings"):
    config.add_section("PEsettings")

# 写入配置
with open(config_path, "w") as configfile:
    config.write(configfile)


# 实时获取窗口大小
def on_resize(event):
    width = event.width
    height = event.height
    config.set("PEsettings", "width", str(width))
    config.set("PEsettings", "height", str(height))
    with open(config_path, "w") as configfile:
        config.write(configfile)


# 打开文件
def open_markdown():
    # 打开文件对话框并获取选择的文件路径
    file_path = filedialog.askopenfilename(filetypes=[("Markdown files", "*.md")])

    if file_path:
        # 读取文件内容
        with open(file_path, "r", encoding="utf-8") as file:
            content = file.read()

        # 解析文件内容
        lines = content.split("\n")
        author, title, content, categories, tags, image_path, alt, pin_status = (
            "",
            "",
            "",
            [],
            [],
            "",
            "",
            True,
        )
        in_metadata = False
        for line in lines:
            line = line.strip()
            if line.startswith("---"):
                in_metadata = not in_metadata
                continue
            if in_metadata:
                if line.startswith("author:"):
                    author = line.split(":")[1].strip()
                elif line.startswith("title:"):
                    title = line.split(":")[1].strip()
                elif line.startswith("image_path:"):
                    image_path = line.split(":")[1].strip().strip('"')
                elif line.startswith("image:"):
                    image_info = line.split(":")[1]
                    if image_info:
                        image_info_dict = yaml.safe_load(image_info)
                        if image_info_dict and "path" in image_info_dict:
                            image_path = image_info_dict["path"].strip('"')
                            if "alt" in image_info_dict:
                                alt = image_info_dict["alt"].strip('"')
                elif line.startswith("pin:"):
                    pin_status = line.split(":")[1].lower() == "true"
                elif line.startswith("categories:"):
                    categories_str = line.split(":")[1].strip().strip("[]")
                    categories = [cat.strip() for cat in categories_str.split(",")]
                elif line.startswith("tags:"):
                    tags_str = line.split(":")[1].strip().strip("[]")
                    tags = [tag.strip() for tag in tags_str.split(",")]

            else:
                content += line + "\n"

        # 检查输入框是否已经包含内容，如果有内容则询问用户是否覆盖
        if (
            author_entry.get()
            or title_entry.get()
            or content_text.get("1.0", tk.END).strip()
            or categories_text.get("1.0", tk.END).strip()
            or tags_text.get("1.0", tk.END).strip()
        ):
            confirm = messagebox.askyesno("覆盖确认", "输入框中已有内容，是否覆盖？")
            if not confirm:
                return

        # 删去多余的换行符
        while content.startswith("\n"):
            content = content[1:]
        # 将解析后的内容填充到对应的输入框中
        author_entry.delete(0, tk.END)
        author_entry.insert(0, author)

        title_entry.delete(0, tk.END)
        title_entry.insert(0, title)

        content_text.delete("1.0", tk.END)
        content_text.insert("1.0", content)

        categories_text.delete("1.0", tk.END)
        categories_text.insert("1.0", "\n".join(categories))

        tags_text.delete("1.0", tk.END)
        tags_text.insert("1.0", "\n".join(tags))

        # 处理图片相关字段（假设您有对应的文本框或变量）
        img_path_entry.delete(0, tk.END)
        img_path_entry.insert(0, image_path)

        img_alt_entry.delete(0, tk.END)
        img_alt_entry.insert(0, alt)

        # 创建一个 IntVar 对象来表示是否置顶，并将其值设置为解析得到的 pin 状态
        option_pin = tk.IntVar(value=int(pin_status))


def save_markdown():
    # 获取输入框中的内容
    author = author_entry.get()
    title = title_entry.get()
    content = content_text.get("1.0", "end-1c")
    selected_option_pin = option_pin.get()
    image_path = img_path_entry.get()
    image = image_entry.get()
    alt = img_alt_entry.get()

    if image != "":
        if alt != "":
            imageINFO = (
                'image:\n  path: "' + image_path + '"\n  ' + 'alt: "' + alt + '"\n'
            )
        else:
            imageINFO = 'image:\n  path: "' + image_path + '"\n'
    else:
        imageINFO = ""

    if image_path != "":
        image_pathINFO = 'image_path: "' + image_path + '"\n'
    else:
        image_pathINFO = ""

    if selected_option_pin == 0:
        pinINFO = "pin: true\n"
    else:
        pinINFO = ""

    if author == "":
        authorINFO = ""
    else:
        authorINFO = "author: " + author + "\n"

    # 替换标题中的空格为短横线
    title = title.replace(" ", "-")

    if title != "":
        categories = categories_text.get("1.0", "end-1c").strip()
        tags = tags_text.get("1.0", "end-1c").strip()

        if categories != "":
            if len(categories.split("\n")) <= 2:
                categoriesINFO = (
                    "categories: [" + ", ".join(categories.split("\n")) + "]\n"
                )
            else:
                messagebox.showerror("警告", "分类应不超过2个")
                return
        else:
            categoriesINFO = ""

        if tags != "":
            tagsINFO = "tags: [" + ", ".join(tags.split("\n")) + "]\n"
        else:
            tagsINFO = ""

        # 获取当前系统时间并格式化
        current_time = datetime.now().strftime("%Y-%m-%d")
        current_timeForMD = datetime.now().strftime("%Y-%m-%d %H:%M:%S +0800")

        # 拼接Markdown格式的内容
        markdown_content = f"---\n{authorINFO}title: {title}\ndata: {current_timeForMD}\n{image_pathINFO}{categoriesINFO}{tagsINFO}{pinINFO}{imageINFO}---\n\n{content}"

        # 保存为Markdown文件
        filename = f"{current_time}-{title}.md"

        # 获取当前脚本所在目录的上级目录作为目标目录
        script_dir = os.path.dirname(os.path.abspath(__file__))
        target_dir = os.path.join(script_dir, "_posts")

        # 确保目标目录存在
        if not os.path.exists(target_dir):
            os.makedirs(target_dir)

        # 构建完整的输出文件路径
        filename = os.path.join(target_dir, f"{current_time}-{title}.md")

        # 写入文件
        with open(filename, "w", encoding="utf-8") as file:
            file.write(markdown_content)

        messagebox.showinfo("提示", f"Markdown文档 '{filename}' 已保存成功！")
    else:
        messagebox.showerror("警告", "缺少标题")


# 创建GUI窗口
root = tk.Tk()
root.title("文章编辑器-[PE]PostsEdit")

width = config.get("BCsettings", "width", fallback="500")
height = config.get("BCsettings", "height", fallback="500")
root.bind("<Configure>", on_resize)
root.geometry(width + "x" + height)
root.minsize(200, 400)

# 输入框、按钮和文本框
open_button = tk.Button(root, text="打开文件", command=open_markdown)
open_button.grid(row=9, column=0, padx=5, pady=5, sticky="w")

author_label = tk.Label(root, text="*作者:")
author_label.grid(row=0, column=0, padx=5, pady=5, sticky="w")
author_entry = tk.Entry(root)
author_entry.grid(row=0, column=1, padx=5, pady=5, sticky="we")

# 创建一个 IntVar 对象来跟踪用户的选择
option_pin = tk.IntVar()

# 创建一个 Label 显示“是否置顶：”
IFpin_label = tk.Label(root, text="*是否置顶:")
IFpin_label.grid(row=1, column=0, padx=5, pady=5, sticky="w")
# 定义一个 Frame 来容纳并排的单选按钮，并设置其样式
options_frame = tk.Frame(root)
options_frame.grid(row=1, column=1, columnspan=2, padx=5, pady=5, sticky="w")
# 在 options_frame 中创建“是”和“否”单选按钮
y_pin = tk.Radiobutton(options_frame, text="是", variable=option_pin, value=0)
n_pin = tk.Radiobutton(options_frame, text="否", variable=option_pin, value=1)
# 将两个单选按钮水平居中放置在 options_frame 内
y_pin.pack(side=tk.LEFT, padx=(0, 10))
n_pin.pack(side=tk.LEFT)
# 初始化 option_pin 的值
option_pin.set(1)

img_path_label = tk.Label(root, text="*图片路径:")
img_path_label.grid(row=2, column=0, padx=5, pady=5, sticky="w")
img_path_entry = tk.Entry(root)
img_path_entry.grid(row=2, column=1, padx=5, pady=5, sticky="we")
img_path_entry.insert(0, "/assets/img/")

image_label = tk.Label(root, text="*封面图:")
image_label.grid(row=3, column=0, padx=5, pady=5, sticky="w")
image_entry = tk.Entry(root)
image_entry.grid(row=3, column=1, padx=5, pady=5, sticky="we")

img_alt_label = tk.Label(root, text="*描述:")
img_alt_label.grid(row=4, column=0, padx=5, pady=5, sticky="w")
img_alt_entry = tk.Entry(root)
img_alt_entry.grid(row=4, column=1, padx=5, pady=5, sticky="we")

categories_label = tk.Label(root, text="分类:")
categories_label.grid(row=5, column=0, padx=5, pady=5, sticky="w")
categories_text = tk.Text(root, height=2, width=50)
categories_text.grid(row=5, column=1, padx=5, pady=5, sticky="we")

tags_label = tk.Label(root, text="标签:")
tags_label.grid(row=6, column=0, padx=5, pady=5, sticky="w")
tags_text = tk.Text(root, height=3, width=50)
tags_text.grid(row=6, column=1, padx=5, pady=5, sticky="we")

title_label = tk.Label(root, text="标题:")
title_label.grid(row=7, column=0, padx=5, pady=5, sticky="w")
title_entry = tk.Entry(root)
title_entry.grid(row=7, column=1, padx=5, pady=5, sticky="we")

content_label = tk.Label(root, text="内容:")
content_label.grid(row=8, column=0, padx=5, pady=5, sticky="nw")
content_text = tk.Text(root, height=10, width=30)
content_text.grid(row=8, column=1, padx=5, pady=5, sticky="nsew")

save_button = ttk.Button(
    root, text="保存为Markdown", command=save_markdown, bootstyle="success"
)
save_button.grid(row=9, column=1, padx=5, pady=5, sticky="we")

# 设置网格布局权重
root.columnconfigure(1, weight=1)
root.rowconfigure(8, weight=1)

# 运行GUI
root.mainloop()
```