# 利用python爬取网站资源的大致步骤

#### 1.设定爬取目标
- 目标网站：（以博客为例子）
- 目标数据：所有文章的标题连接标签
#### 2.分析目标网站
- 待爬取页面
- 待爬取数据：HTML中的h2 class=entry-title下的超链接标题和链接，标签列表。
#### 3.批量下载HTML
- 使用requests库进行下载
#### 4.实现HTML解析，得到目标数据
- 使用BeautifulSoup库进行解析
#### 5.将数据结果储存
- 可以使用json.dumps把数据序列化库存




# 利用Python爬取网站资源完整指南

## 一、准备工作
### 1. 安装必要库
```bash
pip install requests beautifulsoup4 pandas selenium webdriver-manager
```

### 2. 导入基础模块
```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
from time import sleep
import random
```

## 二、基础爬虫流程
### 1. 发送HTTP请求
```python
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
}

response = requests.get('https://example.com', headers=headers)
print(response.status_code)  # 200表示成功
```

### 2. 解析HTML内容
```python
soup = BeautifulSoup(response.text, 'html.parser')

# 提取所有链接
links = [a['href'] for a in soup.find_all('a', href=True)]

# 提取特定元素
titles = [h1.text.strip() for h1 in soup.select('h1.article-title')]
```

### 3. 数据存储
```python
data = {'Title': titles, 'Link': links}
df = pd.DataFrame(data)
df.to_csv('result.csv', index=False)
```

## 三、应对反爬机制
### 1. 请求头伪装
```python
headers = {
    'User-Agent': random.choice(user_agent_list),  # 准备多个UA轮换
    'Referer': 'https://www.google.com/',
    'Accept-Language': 'en-US,en;q=0.9'
}
```

### 2. 频率控制
```python
sleep(random.uniform(1, 3))  # 随机延时
```

### 3. 代理IP使用
```python
proxies = {
    'http': 'http://10.10.1.10:3128',
    'https': 'http://10.10.1.10:1080'
}
response = requests.get(url, proxies=proxies)
```

## 四、动态网页处理（Selenium）
```python
from selenium import webdriver
from webdriver_manager.chrome import ChromeDriverManager

driver = webdriver.Chrome(ChromeDriverManager().install())
driver.get('https://example.com')

# 执行JavaScript
driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")

# 获取动态内容
dynamic_content = driver.find_element_by_css_selector('.load-more-content').text
driver.quit()
```

## 五、数据清洗与存储
### 1. 数据清洗示例
```python
def clean_data(text):
    return text.replace('\n', '').replace('\t', '').strip()
```

### 2. 数据库存储（SQLite）
```python
import sqlite3

conn = sqlite3.connect('data.db')
df.to_sql('articles', conn, if_exists='replace', index=False)
conn.close()
```

## 六、遵守爬虫规范
1. 检查`robots.txt`：访问`https://example.com/robots.txt`
2. 限制请求频率（建议>2秒/次）
3. 不爬取敏感数据（个人隐私、商业机密）
4. 设置爬虫标识：在headers中添加
```python
headers['X-Crawler-Contact'] = 'your@email.com'
```

## 七、完整示例代码
```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import time

def crawler(url):
    headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)...'}
    
    try:
        response = requests.get(url, headers=headers)
        response.raise_for_status()
        
        soup = BeautifulSoup(response.text, 'lxml')
        items = []
        
        for article in soup.select('.article-list > li'):
            title = article.find('h2').text.strip()
            date = article.find('time')['datetime']
            items.append({'title': title, 'date': date})
            
        pd.DataFrame(items).to_csv('news.csv', index=False)
        print(f"成功抓取{len(items)}条数据")
        
    except Exception as e:
        print(f"抓取失败: {str(e)}")

if __name__ == '__main__':
    crawler('https://news.example.com')
    time.sleep(2)
```

## 八、进阶技巧
- 使用Scrapy框架处理大型项目
- 部署分布式爬虫（Scrapy-Redis）
- 处理验证码（OCR识别/打码平台）
- 使用Headless Browser（无头模式）
- 监控爬虫运行（Prometheus + Grafana）

## 九、法律风险提示
1. 遵守《网络安全法》相关规定
2. 不破解网站防护措施
3. 不进行商业性批量抓取
4. 控制数据抓取量（<网站内容20%）

> 推荐工具：Postman（API测试）、Charles（抓包分析）、Scrapy（专业爬虫框架）


---
# 依赖库的使用

## 一、urllib/urllib2

### 1. 基本定义
- Python标准库HTTP请求模块
- urllib2在Python2中更强大，Python3合并到urllib包
- 包含四个模块：
  - `urllib.request`：请求处理
  - `urllib.error`：异常处理
  - `urllib.parse`：URL解析
  - `urllib.robotparser`：robots.txt解析

### 2. 核心方法
```python
from urllib import request, parse

# GET请求
response = request.urlopen('http://www.example.com')
print(response.read().decode('utf-8'))

# POST请求
data = parse.urlencode({'key1': 'value1', 'key2': 'value2'}).encode()
req = request.Request('http://www.example.com/post', data=data)
response = request.urlopen(req)
```

### 3. 技术要点
- 代理设置：
```python
proxy = request.ProxyHandler({'http': 'http://proxy.example.com:8080'})
opener = request.build_opener(proxy)
request.install_opener(opener)
```

- 请求头伪装：
```python
req = request.Request(url)
req.add_header('User-Agent', 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)')
```

### 4. 应用场景
- 基础HTTP请求
- 遵循robots.txt规范的爬虫
- URL参数编码/解码

---

## 二、requests

### 1. 基本定义
- 第三方HTTP客户端库
- 支持HTTP/HTTPS协议
- 人性化API设计（更Pythonic）

### 2. 核心方法
```python
import requests

# GET请求
response = requests.get(url, params={'key':'value'}, headers=headers)

# POST请求
response = requests.post(url, data={'key':'value'}, files=files)

# 会话保持
session = requests.Session()
session.get(url)  # 自动保持cookies
```

### 3. 高级功能
- 文件上传：
```python
files = {'file': open('test.jpg', 'rb')}
requests.post(url, files=files)
```

- SSL验证禁用：
```python
requests.get(url, verify=False)
```

- 超时设置：
```python
requests.get(url, timeout=5)
```

### 4. 应用场景
- RESTful API交互
- 带Cookie保持的登录操作
- 大数据量文件下载（流式传输）

---

## 三、Beautiful Soup

### 1. 基本定义
- HTML/XML解析库
- 支持多种解析器（lxml/html5lib）
- 自动处理编码转换

### 2. 核心方法
```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(html_doc, 'lxml')

# 查找元素
soup.find('div', class_='content')
soup.select('a[href^="http"]')

# 数据提取
links = [a['href'] for a in soup.find_all('a')]
text = soup.get_text()
```

### 3. 解析器对比
| 解析器     | 速度   | 依赖          | 适用场景         |
|------------|--------|---------------|------------------|
| Python标准 | 慢     | 无            | 简单HTML解析     |
| lxml       | 快     | lxml库        | 复杂页面处理     |
| html5lib   | 最慢   | html5lib库    | 解析不规范HTML   |

### 4. 应用场景
- 网页数据清洗
- 多层级嵌套数据提取
- 动态生成内容的静态解析

---

## 四、Selenium

### 1. 基本定义
- 浏览器自动化工具
- 支持主流浏览器（Chrome/Firefox/Edge）
- 可处理JavaScript渲染页面

### 2. 环境配置
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service

service = Service('/path/to/chromedriver')
driver = webdriver.Chrome(service=service)
```

### 3. 核心操作
```python
# 元素定位
driver.find_element(By.ID, 'username').send_keys('admin')
driver.find_element(By.XPATH, '//button[@type="submit"]').click()

# 页面交互
driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")

# 等待策略
from selenium.webdriver.support.ui import WebDriverWait
WebDriverWait(driver, 10).until(
    EC.presence_of_element_located((By.CLASS_NAME, 'result'))
```

### 4. 高级技巧
- 无头模式：
```python
options = webdriver.ChromeOptions()
options.add_argument('--headless')
```

- 文件下载配置：
```python
prefs = {'download.default_directory': '/path/to/save'}
options.add_experimental_option('prefs', prefs)
```

- 浏览器指纹伪装：
```python
options.add_argument("--disable-blink-features=AutomationControlled")
```

### 5. 应用场景
- 动态网页数据抓取
- 自动化测试
- 网页截图/PDF生成
- 验证码破解（配合OCR）

---

## 五、综合应用案例
### 1. 全流程数据采集
```python
import requests
from bs4 import BeautifulSoup
from selenium import webdriver

# 使用requests获取初始页面
response = requests.get('https://example.com/list')
soup = BeautifulSoup(response.text, 'lxml')

# 提取动态加载链接
detail_links = [a['href'] for a in soup.select('a.detail-link')]

# 使用Selenium处理动态内容
driver = webdriver.Chrome()
for link in detail_links:
    driver.get(link)
    dynamic_content = driver.find_element(By.CLASS_NAME, 'content').text
    # 数据存储处理...
driver.quit()
```

### 2. 反爬对抗策略
- 请求频率控制：`time.sleep(random.uniform(1,3))`
- IP轮换：结合代理中间件
- 验证码处理：`pytesseract` OCR识别
- 指纹伪装：修改WebDriver属性

---

## 六、工具选型建议
| 需求场景               | 推荐工具               | 原因说明                   |
|------------------------|------------------------|----------------------------|
| 简单API请求           | requests              | 接口简洁，开发效率高       |
| 基础静态页面解析       | urllib + BeautifulSoup | 无需额外依赖               |
| 复杂动态页面交互       | Selenium              | 完整浏览器环境支持         |
| 大规模分布式爬虫       | Scrapy                | 内置异步处理和中间件机制   |

> **注意事项**：
> 1. 遵守目标网站robots.txt协议
> 2. 设置合理的请求间隔（建议>2秒）
> 3. 使用`try-except`处理网络异常
> 4. 敏感数据需进行脱敏处理
```