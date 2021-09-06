# 爬虫入门及selenium教程(python)
## 1.概论
爬虫是在网页上收集信息的程序，一般由两个部分组成，即爬取网页和解析网页。

爬虫的难点有二：1.大规模，高效率爬取 2.对动态网页或反爬机制进行针对。前者需要利用scrapy等框架进行多线程处理，本文主要介绍的selenium则适用于后者。
## 2.最简单的爬虫
	import requests
	import re
	import os
	import sys
	def get_downloadurl():
		#写你想要的美剧所在的地址
		resource_url = "http://yyetss.com/detail-4800.html"
		r = requests.get(resource_url, timeout=20)
		data = r.content
		data = data.decode('utf-8')
		download_list = re.findall(r'ed2k://\|file\|.{,200}\|/', data)
		if download_list:
			with open('result.txt', 'w') as f:
				for i in download_list:
					f.write('%s\n' % (i))
				print('Save as result.txt')
		else:
			print('No Resource')

