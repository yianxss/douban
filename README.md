import requests
from bs4 import BeautifulSoup
import random
import time
import re
import csv
def get_bookname_list(url):
    headers={
    'Host':'book.douban.com',
    'Referer':'https://book.douban.com/tag/',
    'User-Agent':'Mozilla/5.0 (iPhone; CPU iPhone '
                 'OS 9_1 like Mac OS X) AppleWebKit/601.1.46 '
                 '(KHTML, like Gecko) Version/9.0 Mobile/13B143 Safari/601.1',
    'Accept':'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'Accept-Encoding':'gzip, deflate, sdch, br',
    'Accept-Language':'zh-CN,zh;q=0.8',
    'Cache-Control':'max-age=0',
    'Connection':'keep-alive',
    'Cookie':'bid=CDY5tquin9c; gr_user_id=5000dd64-5383-4752-b3f5-e1115f252408; '
             'll="118159"; dbcl2="14001580:QX6iG+LG4lo"; ps=y; ap=1; ck=3-Db; viewed='
             '"20429677_1084336"; _vwo_uuid_v2=E45E381696B46DFE88FD60B36619BD1C|ca84f0'
             '8a99dc4c32f088d7d46482c1f9; _pk_ref.100001.3ac3=%5B%22%22%2C%22%22%2C1475'
             '067971%2C%22https%3A%2F%2Fwww.douban.com%2F%22%5D; _pk_id.100001.3ac3=83458e'
             '45e14793b9.1474822646.8.1475067971.1475063808.; _pk_ses.100001.3ac3=*; __u'
             'tmt_douban=1; __utma=30149280.27001266.1474822634.1475063811.1475067971.8;'
             ' __utmb=30149280.1.10.1475067971; __utmc=30149280; __utmz=30149280.147482'
             '2634.1.1.utmcsr=baidu|utmccn=(organic)|utmcmd=organic; __utmt=1; __utma=81'
             '379588.1611208420.1474822645.1475063811.1475067971.8; __utmb=81379588.1.10.'
             '1475067971; __utmc=81379588; __utmz=81379588.1474981390.3.3.utmcsr=douban.co'
             'm|utmccn=(referral)|utmcmd=referral|utmcct=/'
       }
    r=requests.get(url,headers=headers).text

    print(r[-2])
    return re.findall(r'<a href="/tag/(.*)">',r)

def get_booklist(all_urls):
    csv_f=open("out.csv",'w',newline='')
    spamwriter=csv.writer(csv_f,dialect='excel')
    spamwriter.writerow(['书名','作者','评分','出版日期','价格'])
    headers={
    'Accept':'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'Connection':'keep-alive',
    'Host':'book.douban.com',
    'Referer':'https://book.douban.com/tag/%E5%B0%8F%E8%AF%B4',
    'Cookie':'bid=CDY5tquin9c; gr_user_id=5000dd64-5383-4752-b3f5-e1115f252408; '
             'll="118159"; dbcl2="14001580:QX6iG+LG4lo"; ps=y; ap=1; ck=3-Db; _vwo'
             '_uuid_v2=E45E381696B46DFE88FD60B36619BD1C|ca84f08a99dc4c32f088d7d46482'
             'c1f9; _',
    'User-Agent':'Mozilla/5.0 (iPhone; CPU iPhone OS 9_1 like Mac OS X) AppleWe'
                 'bKit/601.1.46 (KHTML, like Gecko) Version/9.0 Mobile/13B143 Safari/601.1'
}
    for fen_url in all_urls:
        for i in range(100):
            time.sleep(random.randrange(1,5))
            url='https://book.douban.com/tag/'+fen_url+'?start='+str(i*20)+'&type=T#!/i!/ckDefault'
            r=requests.get(url,headers=headers)
            print(r.status_code)
            print(r.text[-10])
            soup=BeautifulSoup(r.text,'lxml')
            all_datas=soup.find_all('li',class_="subject-item ckd-item")
            for data in all_datas:
                try:
                    book_score=eval(data.find('span',class_="rating-star-with-text").span.get('data-rating'))/10
                    book_name=data.find('a', href=True,title=True).get('title')
                    book_other_mes=data.find('div',class_="pub ckd-desc").string.replace('\n','').replace(' ','').split('/')
                    book_writer='_'.join(book_other_mes[:len(book_other_mes)-3])
                    book_year=book_other_mes[-2]
                    book_price=book_other_mes[-1]
                    allwrite=[str(book_name),book_writer,str(book_score),str(book_year),str(book_price)]
                    spamwriter.writerow(allwrite)
                    print('正在下载%s'%fen_url)
                    print(allwrite)
                except:
                    print(book_name)

    csv_f.close()


if __name__ == "__main__":
     all_datas=get_bookname_list('https://book.douban.com/tag/')
     print(all_datas)
     get_booklist(all_datas)
