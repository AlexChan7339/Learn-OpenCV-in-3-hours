# 2-安装opencv包

- 新建虚拟环境

  ```python
  # pycharm terminal
  conda create -n opencv python=3.10
  ```

- 安装opencv包

```python
# pycharm terminal
pip install opencv-python
```

- 验证是否安装成功

  ```python
  import cv2	# CV2 代表计算机视觉
  print("Package Imported")
  ```