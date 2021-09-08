# 爬虫入门及selenium教程(python)

## 1.概论
### 1.1 爬虫是什么
爬虫是在网页上收集信息的程序，一般由两个部分组成，即爬取网页和解析网页。

爬虫的难点有二：1.大规模，高效率爬取 2.对动态网页或反爬机制进行针对。

### 1.2 爬虫的工具包(python)
爬取库：requests(基础)，selenium(操作较复杂，因为模拟浏览器，具有强大的反爬和动态处理能力)

解析库：lxml(xpath)，re(正则表达式)，bs4(对xpath的封装改造，搜索功能较全面，但是不直接支持xpath)。 re可以对任何文本进行处理，lxml和bs4只能处理html和xml，但可以展示出网页的层级结构。

框架：scrapy，它同时集合了爬取和解析的功能，可以用于开发大型爬虫

本文将重点介绍selenium+lxml的组合

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

注1：对于初学者来说，如果没有反爬或复杂需求，可以不学习selenium。 使用requests+lxml/re/bs4或单独使用scrapy已能完成基础需求。

注2：如何看懂html是爬虫的难点，在第6点会对此进行介绍。

## 4.selenium入门
selenium通过创建模拟浏览器的方式进行爬取，不但可以实现登录等动态操作，而且可以规避跳转反爬。

跳转反爬：通过识别跳转行为来判断访问是否正常。requests和selenium的get方法都相当于地址栏跳转，而常规用户会从网站门页不断向里深入，selenium可以很方便地模仿这一行为。

### 4.1 安装selenium
参见教程：https://www.cnblogs.com/lfri/p/10542797.html
,https://www.cnblogs.com/shaosks/p/14857640.html

太长不看版：

1.除了安装selenium包之外，还需要根据自己的浏览器安装驱动，目前selenium只支持Chrome和火狐。我安装的是Chromedriver，以下演示也是使用它。

2.假如不想配置环境变量，需要将驱动同时安装到浏览器目录和python目录下

### 4.2 简单实例：
在百度图片搜索"python"，下滑滚动条加载更多图片，并下载前十张图
```python
from selenium import webdriver
import time
import requests

driver = webdriver.Chrome()#声明浏览器对象

try:
    driver.get("https://image.baidu.com")#相当于地址栏跳转
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

    driver.close()#关闭当前标签页

except Exception as e:
    print (e)
```
程序运行界面如下，selenium会打开一个新的浏览器窗口
![selenium界面](/images/1.png)
### 4.3 selenium基础语句

#### 4.3.1 搜索元素

find_element_by_id

注意：在html中，为避免与js语法冲突，id唯一，因此优先考虑使用id查找元素

find_element_by_xpath

find_element_by_css_selector

熟练掌握xpath或css_selector的其中一个，在爬取和解析时，它们都能起到作用！

find_element_by_name

find_element_by_tag_name

find_element_by_class_name

#### 4.3.2 元素

网页元素在selenium中被划为WebElement类，有以下常用属性和方法:

element.tag_name #标签名，如 'a'表示<a>元素

element.text #该元素内的文本，例如<span>hello</span>中的'hello'

element.get_attribute(x)# 该元素x属性的值

#### 4.3.3 模拟浏览器操作

鼠标事件，可参见链接：https://www.cnblogs.com/youngleesin/p/10449356.html

其中重要的鼠标事件摘录如下：

element.click()#点击

element.send_keys()#输入

ActionChains(driver).move_by_offset(xoffset,yoffset).perform()#移动鼠标，有些网页的弹窗需要我们做移开鼠标动作

ActionChains(driver).drag_and_drop_by_offset(source, xoffset, yoffset)#拖拽，多用于自动解验证码。拖滚动条用js语句更方便

其他功能

driver.get()#地址栏跳转

driver.execute_script("window.scrollTo(0,document.body.scrollHeight)")#执行js语句，这句的意思是将滚动条拖至底部

driver.switch_to.window(driver.window_handles[-1])#切换至最新标签页

#### 4.4.4 保存页面源代码
```python
file=open(path,'w',encoding='utf-8')
file.write(repr(driver.page_source))#repr() 函数将对象转化为供解释器读取的形式
file.close()
```

## 5.selenium的高级用法
### 5.1 标签页管理
selenium通过driver.switch_to.window()来切换所操作的标签页。假如一个行动打开了新的标签页，在旧页面无法继续操作新页面元素，必须先switch过去

driver.window_handles属性返回一个列表，列出目前打开的所有标签页，以时间顺序排列，故 driver.switch_to.window(driver.window_handles[-1]) 可以切换至最新标签页

driver.close()方法关闭当前标签页，driver.quit()方法关闭所有打开的标签页，假如所有标签页都被关闭，那么程序就会停止

#### 5.1.1 锚点
一个小技巧:在遍历网页的子页时，可以将目录页保存下来，以便访问完子窗口后回到母窗口
```python
anchor = driver.window_handles[-1]#将当前标签页设为锚点

driver.switch_to.window(anchor)#返回保存的标签页
```
### 5.2 反爬！！
#### 5.2.1 常见反爬机制
前文已经介绍过一些，现在做一个总结

检测异常行为：如反复访问，地址栏跳转到网站深处，频繁访问等。可通过构建代理ip池以及优化爬取逻辑等规避

加密网站信息：一种加密方法是，将一些字符串转为图片，这些图片在html中会变为乱码。碰到这些，需要具体问题具体分析

检测selenium：禁止selenium等自动控制软件访问，可通过设置以及js调整

验证码：可通过设置程序自动解验证码或人工守在爬虫旁解决

检测请求头：header中的Cookie、Referer、User-Agent都是检查点，在requests中需要专门设置，selenium一般不会遇到这个问题

#### 5.2.2 selenium反爬
通过调整浏览器选项，提供稳定运行环境并进行反爬。

一般来说，在声明浏览器前后，加上这堆东西，就不会被识别出来，此时还能威胁到我们的只剩异常访问和验证码。验证码的解决方案会在下一点中介绍，但异常访问只能靠优化爬取或者代理池来解决。
```python
options = webdriver.ChromeOptions()

#稳定运行
options.add_argument('-enable-webgl')#解决 GL is disabled的问题
options.add_argument('--no-sandbox')  # 解决DevToolsActivePort文件不存在的报错
options.add_argument('--disable-dev-shm-usage') 
options.add_argument('--ignore-gpu-blacklist') 
options.add_argument('--allow-file-access-from-files') 

#反爬
options.add_experimental_option("excludeSwitches", ["enable-automation"])# 模拟真正浏览器
options.add_experimental_option('useAutomationExtension', False)

driver = webdriver.Chrome(options=options)#声明浏览器
#模拟真正浏览器
driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {
  "source": """
    Object.defineProperties(navigator,{webdriver:{get:() => false}});
  """
})
```
### 5.3 动态操作
#### 5.3.1 滚动条到底有多长？
大家肯定见过这样的网站:它不会一次加载完所有信息，需要我们拖滚动条才能继续加载。然而，这个滚动条，到底拖多久才是个头？

我们以百度图片搜索"传承换心"为例，尝试将它的滚动条拖到底端，加载所有图片
```python
from selenium import webdriver
import time

driver = webdriver.Chrome()#声明浏览器对象

try:
    driver.get("https://image.baidu.com")#相当于地址栏跳转
    box = driver.find_element_by_id('kw')#找到输入框
    box.click() 
    box.send_keys("传承换心")#先点一下，再输入内容
    button=driver.find_element_by_xpath("//input[@type='submit']")#找到按钮"百度一下"
    button.click()#按按钮
    
    js='window.scrollTo(0,document.body.scrollHeight)' #下滑到底部
    js2='return document.documentElement.scrollHeight' #检测当前滑动条位置

    #对于内嵌滚动条(不是网页边缘，而是网页里面可以拖动的东西)，需要用getElements把这个元素找出来，再和刚才做一样的操作
    #js='document.getElementsByClassName("unifycontainer-two-wrapper")[0].scrollTop=1000000' 
    #js2='return document.getElementsByClassName("unifycontainer-two-wrapper")[0].scrollHeight'
    
    height=-1
    now=driver.execute_script(js2)
    #当滑动条位置不再被“下滑到底部”这一行为影响，说明滑动条真的到了底部
    while height!=now:
        height=now
        driver.execute_script(js)
        time.sleep(1)
        now=driver.execute_script(js2)
    
    time.sleep(5)#对于想要看结果的程序，在最后设置一下暂停
    driver.close()
    #当然，也可以直接删掉close语句，让浏览器一直开着……

except Exception as e:
    print (e)

```
#### 5.3.2 验证码
验证码，据我所知有三个对策

1.使用cookie避免输入验证码

参见链接：https://blog.csdn.net/weixin_46457203/article/details/105857918

2.自动识别验证码

验证码种类多样，难以一一列举，和网站加密一样，需要具体问题具体分析

3.最为暴力的人工法

前两种方法泛用性不强，自动识别需要自己设定程序，而有些网站无视cookie，每次登陆都需要验证码。

我使用的方法是在需要输入验证码的环节设置一个time.sleep()。在time.sleep期间，我们可以对selenium打开的浏览器进行操作，比如人工输入验证码，跳转几个页面等。只要在sleep结束后，你去到的网页能够接上后续程序，就不会有任何问题。

许多大型爬虫机构依然使用人工法，因为人工法对劳动力质量和数量要求都不高。
### 5.4 黑科技:自动录制脚本
对于新手来说，怎么将浏览器操作转化为代码，或许还有难度。我们可以利用selenium IDE浏览器插件，录制在浏览器上的操作，并自动生成代码。

在火狐 更多工具->面向开发者的拓展 一栏可以很方便下载，但是Chrome的插件似乎需要翻墙，难顶。

参考链接：https://www.cnblogs.com/shuaijie/articles/4552012.html

### 5.5 进阶教程
这篇文章对selenium的介绍更全面详细，适合想要精益求精的大佬
https://blog.csdn.net/qq_44695727/article/details/107334574

## 6.html入门+lxml与xpath的使用
### 6.1 查看网页源代码
在浏览器中，随便打开一个网页，点击右键，会看见"检查"选项。点击这一项，即可查看网页源代码。假如把鼠标移动到网页的某个元素上，再选择检查，可以方便地查看到该元素在html中的位置。下图查看的是"百度一下"按钮。

![检查](/images/2.PNG)

### 6.2 很简略的html语法
在html中，尖括号<>叫做标签。标签可以成对出现，如head(标题，样式等)，body(主体内容)等，也可以单个出现，如input(输入控件)，img(图片)等。标签之间可以嵌套，一对标签之中可以夹着任意对标签，这构成了html的层级关系。如下图中，input就是span的子节点。

标签具有属性，以"A"=B的格式书写，说明标签有一个叫A的属性，它的值是B。如下图中，input标签的value属性值为"百度一下"。

注：html中，id属性取值唯一，也就是说id能唯一确定一个标签(假如该标签有这个属性)

![html](/images/3.PNG)

关于html的详细教程，可以参考https://www.w3school.com.cn/html/index.asp

### 6.3 xpath
XPath 是一门在 XML 文档中查找信息的语言，详细教程可参考https://www.cnblogs.com/zhangxinqi/p/9210211.html

这里用例子说明一些常用语法

//img[@title]/@title

查找所有有title属性的img节点，并获取它们的title值。 //表示对当前节点以下所有节点进行搜索，/表示搜索当前节点的子节点，或用于获取属性

//img[@title='PKU']

查找所有title属性为"PKU"的img节点

//@title

查找xml中所有title属性的值。而 /@title 表示查看根节点的title属性，大部分情况下会返回空集，因为根节点无此属性

//img/img2

查找所有img节点下的img2子节点

//img/img2[1]

查找所有img节点下的第一个img2子节点，注意xpath的index不是从0而是从1开始

(//img/img2)[1]

查找所有img节点下的img2子节点，并取第一个

//img/node()

node()，模糊匹配，表示任意子节点。这句意思为查找所有img节点的所有子节点

//a/text()

查找所有a节点的内容

### 6.4 lxml
lxml是利用xpath查找元素的解析库。节点在lxml中被定义为Element类(和selenium的webElement相似)，对于一些难以用xpath表达的复杂需求，可以先用xpath锁定该节点，再调用element类的属性和方法进行处理

关于element类的方法属性，参见官方文档：https://lxml.de/api/lxml.etree._Element-class.html

以下是一个简单的例子，以下为需要处理的html，节选自大众点评目录页，我们尝试把"4.54"这个数字拿出来
```
<div class="nebula_star">
<div class="star_icon">     
<span class="star star_45 star_sml"></span>
<span class="star star_45 star_sml"></span>
<span class="star star_45 star_sml"></span>
<span class="star star_45 star_sml"></span>
<span class="star star_45 star_sml"></span>
</div>
<div class="star_score score_45  star_score_sml">4.54</div>
</div>
```
```python
path='demo.html'
html_text = open(path, 'r', encoding='utf-8').read()
r = html.etree.HTML(html_text)#读取html并转为etree

#方法一，直接使用xpath
scorelis=r.xpath("//div[@class='nebula_star']/div[2]/text()")

#但是在实践中，我发现有的<div class="nebula_star">中，并不是都有评分，假如按照方法一，出现缺漏，不知道是哪个漏了，因此最后选用的是以下方法
#方法二
scorelis=[]
scorenode=r.xpath("//div[@class='nebula_star']")
for s in scorenode:
    lis=s.findall("div")
    if len(lis)==1:
        scorelis.append('nan')
    else:
        scorelis.append(lis[1].text)
#这样，缺少的分数会被转化为nan
```
### 7.总结
爬虫是入门容易精通难的技术，在数据分析领域常常用到(当然，一般都是萌新负责的dirty work)。我才疏学浅，权当抛砖引玉，如有错误，还请多多指正。

### 8.附：美团系列字体加密破解方法
它有专门的字体包，用图片替换了部分文字，下载对应字体包即可。

参见链接：https://www.icode9.com/content-4-803615.html