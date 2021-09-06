# 爬虫入门及selenium教程(python)

## 1.概论
爬虫是在网页上收集信息的程序，一般由两个部分组成，即爬取网页和解析网页。

爬虫的难点有二：1.大规模，高效率爬取 2.对动态网页或反爬机制进行针对。本文重点介绍的selenium是处理第二点的好方法。

## 2.最简单的爬虫
用requests进行爬取，re模块进行解析。

爬取T大经管学院官网，看看它一共有多少个系(该校官网有反爬，不适合作为新手教程)
```python
import requests
import re
    
resource_url = "http://www.sem.tsinghua.edu.cn/"
r = requests.get(resource_url, timeout=20)#爬取网页
data = r.content
data = data.decode('utf-8')
download_list = re.findall(r'href="/\w*/">(\w+)</a>', data)#解析网页，提取所需字段
#re.findall只捕捉分组信息，即括号内字段！

#保存数据&异常处理
if download_list:
    with open('result.txt', 'w') as f:
        for i in download_list:
            f.write('%s\n' % (i))
        print('Save as result.txt')
else:
    print('No Resource')
```
## 3.增加容错:保存原网页
在上面这段程序中，原网页数据以变量的方式储存。假如程序结束或出现异常，这些数据就会丢失，需要重新爬取。而面对反爬机制强大的网站(美团系列)时，多次重复爬取将会导致封号等严重后果。(建议多在百度这种不会封号的网站练习，再去与反爬机制较量)

另外，即使是老手也没办法保证无需debug就能准确解析所需字段，保存原网页可以节省重复爬取的时间。

在上文程序中添加以下代码，即可将原网页保存。
```python
path='./page.txt'
with open(path,'wb') as f:
    f.write(r.content)
    f.close()
```
打开txt文件并解析
```python
import re

with open(path, "r",encoding='utf-8') as f:    
    data = f.read()   

download_list = re.findall(r'href="/\w*/">(\w+)</a>', data)
print(download_list)
```
另外，网页也可被保存为html格式，根据爬取库和解析库来选择保存为txt还是html。下文介绍selenium+lxml的组合，使用html保存。

注1:常用爬虫解析库有lxml(xpath)，re(正则表达式)以及bs4(css选择以及正则表达式)。re可以对任何文本进行处理，lxml和bs4只能处理html和xml，但可以展示出网页的层级结构。

注2:对于初学者来说，如果没有反爬或复杂需求，可以不学习selenium。 requests+lxml/re/bs4(三者均可)，或是scrapy(一个爬虫框架，包含爬取和解析两部分)已能完成基础爬虫需求。

## 4.selenium入门
selenium通过创建模拟浏览器的方式进行爬取，不但可以实现登录等动态操作，而且可以规避跳转反爬。

跳转反爬：通过识别跳转行为来判断访问是否正常。requests等普通爬虫相当于从空白标签页一下深入到网站内部，而常规用户会从网站门页不断向里深入。

### 安装selenium
参见教程:https://www.cnblogs.com/lfri/p/10542797.html

除了安装selenium包之外，还需要根据自己的浏览器安装驱动，首选chromedriver(Chrome)，其次是geckodriver(火狐)

### 简单实例:
在百度图片搜索"python"，下滑滚动条加载更多图片，并下载前十张图
```python
from selenium import webdriver
import time
import requests

driver = webdriver.Chrome()#声明浏览器对象

try:
    driver.get("https://image.baidu.com")#相当于浏览器上端地址栏跳转
    box = driver.find_element_by_id('kw')#找到输入框
    box.click() 
    box.send_keys("python")#先点一下，再输入内容
    button=driver.find_element_by_xpath("//input[@type='submit']")#找到按钮"百度一下"
    button.click()#按按钮
    time.sleep(2)#在爬虫中多用sleep，而且最好配合random，规避时间反爬(不间隔连续快速操作，有可能被识别为脚本)

    for i in range(3):
        driver.execute_script("window.scrollTo(0,document.body.scrollHeight)")#用js拖动滚动条
        time.sleep(2)

    eles=driver.find_elements_by_xpath("//li[@class='imgitem']")#找到图片们，注意这里的find_elements是复数，因此返回一个元素列表
    for i,e in enumerate(eles[:10]):
        img_data = requests.get(e.get_attribute("data-thumburl")).content#配合requests去到下载地址，用selenium需要反复跳，不方便
        img_path = 'spider_demo/'+str(i)+'.jpg'
        fp=open(img_path,'wb')
        fp.write(img_data)

    driver.close()

except Exception as e:
    print (e)
```

### selenium基础语句

#### 搜索元素

find_element_by_id

注意:在html中，为避免与js语法冲突，id唯一，因此优先考虑使用id查找元素

find_element_by_xpath

熟练掌握xpath或css_selector的其中一个，在爬取和解析时，它们都能起到作用！

find_element_by_name

find_element_by_tag_name

find_element_by_class_name

find_element_by_css_selector

#### 模拟浏览器操作
element.click()

点击

element.send_keys()

输入

driver.get()

地址栏

