# Python 爬虫最常用、最实用的库（全分类 + 用途 + 新手推荐）
我给你整理**最核心、最常用、工作/学习必用**的爬虫库，按功能分类，新手直接照着选就行，不绕弯。

## 一、发送网络请求（爬虫第一步：拿网页）
### 1. **requests**（最经典、最常用、新手必学）
- 用途：**发送 HTTP 请求**，获取网页源代码
- 特点：简单、稳定、90% 爬虫都用它
- 适合：静态网页、接口数据

```python
import requests
resp = requests.get("https://www.baidu.com")
print(resp.text)
```

### 2. **httpx**（requests 升级版）
- 支持 HTTP/2、异步请求
- 写法和 requests 几乎一样，更现代

---

## 二、解析网页（从源码里提取内容）
### 1. **BeautifulSoup4（bs4）**（新手首选）
- 用途：**解析 HTML/XML，轻松提取标题、文字、链接、图片**
- 特点：语法超级简单，容错率高

```python
from bs4 import BeautifulSoup
soup = BeautifulSoup(resp.text, "html.parser")
title = soup.find("title").text
```

### 2. **lxml**（速度最快）
- 配合 bs4 或 xpath 使用
- 大型爬虫必用

### 3. **parsel**（支持 xpath + css 选择器）
- scrapy 框架内置解析器

---

## 三、处理动态网页（JS渲染、滚动加载、登录）
很多国外网站、电商、社交网站**不是直接返回源码**，必须用**模拟浏览器**。

### 1. **Selenium**（最老牌、最稳定）
- 用途：**控制真实浏览器**（Chrome/Edge）
- 适合：需要点击、登录、滑动、JS渲染的网站

```python
from selenium import webdriver
driver = webdriver.Chrome()
driver.get("https://www.google.com")
```

### 2. **Playwright**（微软出品，最强）
- 比 selenium 更快、更稳、自动等待
- 支持自动录屏生成代码
- **现在最推荐的动态爬虫工具**

### 3. **Pyppeteer**
- 无头浏览器，轻量

---

## 四、爬虫框架（大型、批量、工程化爬虫）
### **Scrapy**（爬虫界第一框架）
- 用途：**批量爬取大量数据、分布式爬虫**
- 企业最常用
- 自带并发、去重、爬虫中间件、数据管道

---

## 五、数据提取与处理
### 1. **re**（正则表达式）
- 提取复杂文本、邮箱、手机号、链接

### 2. **json**
- 解析接口返回的 JSON 数据

### 3. **pandas**
- 爬完数据保存到 Excel / CSV

---

## 六、反爬、代理、加解密
### 1. **fake-useragent**
- 随机生成请求头，伪装浏览器

### 2. **proxypool**
- 代理IP池（防封IP）

### 3. **pycryptodome**
- 破解网站加密参数

### 4. **execjs**
- 执行 JS 代码，破解前端加密

---

## 七、异步爬虫（提速神器）
### 1. **aiohttp**
- 异步请求，比 requests 快 10~100 倍
- 适合大批量爬取

### 2. **asyncio**
- Python 内置异步库

---

# 🔥 新手学习路线（最简单、最实用）
1. **requests + bs4** → 静态网页爬虫（90% 场景）
2. **Playwright** → 动态网页、国外网站、登录、JS渲染
3. **Scrapy** → 大型批量爬虫

---

## 你最需要的一句话总结
- **普通爬文章/新闻/列表**：`requests + bs4`
- **爬国外网站、动态加载**：`Playwright`
- **企业批量爬虫**：`Scrapy`

我可以直接给你写**可运行的极简模板代码**，你想爬哪种网站？