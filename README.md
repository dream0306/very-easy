# very-easy or not very easy
# coding:utf-8
from selenium import webdriver
#从selenium到webdriver
from selenium.webdriver.support.ui import WebDriverWait
#x秒内每隔500毫秒扫描1次页面变化
import requests
import re
import os
import time
from bs4 import BeautifulSoup
#爬墙方式
browser = webdriver.Chrome()
# 设置网站等待时间3秒
wait = WebDriverWait(browser, 3)

def get_html(url):
    print('正在爬取...')
    try:
        browser.get(url)
        html=browser.page_source
        if html:
            return html
    except EOFError:
        return None
def get_html(url):
     page = urllib.urlopen(url)
     html = page.read()
    return html

def get_page():
    base_url = 'http://jandan.net/ooxx/' #第一次进入的原始地址
    soup = BeautifulSoup(get_html(base_url), 'lxml')  #解析原始地址
    current_pages=soup.select('.cp-pagenavi .current-comment-page')[0].get_text()[1:-1] #取出当前页面字段
    urllist = []
    for page in range(1,int(current_pages)+1):
        real_url=base_url+'page-'+str(page)+'#comments' #拼出分页地址
        urllist.append(real_url)
    return urllist #返回所有分页地址列表

def mkdir():
    isExists=os.path.exists('./jiandan')
    if not isExists:
        print('创建目录')
        os.makedirs('./jiandan') #创建目录
        os.chdir('./jiandan') #切换到创建的文件夹
        return True
    else:
        print('目录已存在,即将保存！')
        return False

def get_pic():
    for url in get_page():
        mkdir()
        html=get_html(url)
        soup=BeautifulSoup(html,'lxml')
        allimgs=soup.select('div .text p img')
        allimgs=re.findall('src="(.*?)"',str(allimgs))
        download(allimgs)
    print("下载完毕！")

def download(allimgs):
    for img_url in allimgs:
        filename = './jiandan\\'+img_url.split('/')[-1]
        with open(filename,'wb+') as f:
            try:
                f.write(requests.get(img_url).content)
                print('成功下载图片',filename)
            except:
                print('下载失败',filename)

def main():
    # 计算全部下载完需要多长时间
    t = time.time()
    get_pic()
    print(time.time() - t)

if __name__ == '__main__':
    main()
