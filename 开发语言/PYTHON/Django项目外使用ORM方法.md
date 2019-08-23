# Django项目外使用ORM方法



```
import os, sys
BASE_DIR = os.path.dirname(os.path.abspath(__file__))  # 定位到你的django根目录
sys.path.append(os.path.abspath(os.path.join(BASE_DIR, os.pardir)))
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "dj_tasks.settings")  # 你的django的settings文件
```



