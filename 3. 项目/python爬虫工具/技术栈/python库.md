# Python 热门常用库（三级标题·纯整理版）
## 一、Web 开发类
### FastAPI
高性能现代Web接口框架，自动生成接口文档，适合后端API、AI模型部署，标配搭配Pydantic。

### Flask
轻量极简Web框架，自由度高，适合小型项目、个人接口、快速原型开发。

### Django
全能型企业级Web框架，自带后台管理、用户认证、ORM、安全防护，适合大型网站、管理系统。

### Uvicorn
ASGI异步服务器，用来运行FastAPI、异步Web项目。

## 二、数据建模与校验
### Pydantic
独立数据模型校验库，做类型约束、自动类型转换、参数校验、JSON结构化，FastAPI强依赖。

## 三、爬虫与自动化
### Requests
最常用HTTP请求库，发送GET/POST请求，获取网页源码、接口数据。

### BeautifulSoup4
HTML/XML解析库，轻松定位标签、提取文本、链接、属性。

### Playwright
微软出品浏览器自动化库，渲染JS动态页面、模拟点击登录、爬取国外网站。

### Selenium
老牌浏览器自动化工具，用于动态爬虫、WebUI自动化测试。

### Scrapy
企业级爬虫框架，自带并发、去重、调度、数据管道，适合大规模分布式爬取。

## 四、数据分析与可视化
### NumPy
Python数值计算基础库，提供高性能数组、矩阵运算，是Pandas、AI库底层依赖。

### Pandas
数据分析核心库，读写Excel/CSV、数据清洗、筛选分组、统计分析。

### Matplotlib
基础绘图可视化库，绘制折线图、柱状图、散点图等各类图表。

### Seaborn
基于Matplotlib的高级可视化库，画风更好看，适合统计绘图。

## 五、办公文件处理
### openpyxl
读写编辑Excel文件（xlsx格式），做办公自动化、报表生成。

### python-docx
读写Word文档，生成、修改docx文件内容。

## 六、人工智能与机器学习
### Scikit-learn
经典机器学习库，提供分类、回归、聚类、特征工程、模型评估全套算法。

### PyTorch
主流深度学习框架，科研、大模型训练、AI项目首选。

### TensorFlow/Keras
谷歌深度学习框架，适合工业级部署、模型落地。

### transformers
Hugging Face开源库，一键调用bert、GPT等各类预训练大模型。

## 七、数据库操作
### PyMySQL
Python连接操作MySQL数据库。

### SQLAlchemy
经典ORM框架，不用手写SQL，用面向对象方式操作数据库。

### redis-py
Python操作Redis缓存，做缓存、限流、临时数据存储。

## 八、异步网络
### aiohttp
异步HTTP请求库，异步爬虫、异步接口开发，比Requests效率高很多。

## 九、实用工具库
### loguru
极简日志库，替代原生logging，自动分割日志、格式美观。

### tqdm
命令行进度条库，循环任务、文件下载显示进度。

### pyinstaller
将Python代码打包成Windows exe可执行文件，免环境直接运行。

### fake-useragent
随机生成浏览器User-Agent，爬虫反爬伪装用。