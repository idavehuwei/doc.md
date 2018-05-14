from multiprocessing import Pool
import requests
from bs4 import BeautifulSoup
import re

url = 'http://sou.zhaopin.com/jobs/searchresult.ashx?jl=%E6%88%90%E9%83%BD&kw=%E8%BF%90%E7%BB%B4&p=1&isadv=0'

class Job_Search(object):

    def __init__(self,url):
        self.data = requests.get(url).content
        self.soup = BeautifulSoup(self.data, 'lxml')
        self.pages = self.get_pages()
        self.job_data = {}

    def get_pages(self):
        items = self.soup.select("div#newlist_list_content_table > table")   #找到每个职位
        count = len(items) - 1      #计算每页多少个职位
        job_count = re.findall(r"共<em>(.*?)</em>个职位满足条件", str(self.soup))[0] #取到页面中总职位数
        self.pages = (int(job_count) // count) + 1                   #+1是因为int不会取舍,直接丢掉小数点后
        return self.pages

    def get_job(self):
        job_name = self.soup.select("table.newlist > tr > td.zwmc > div > a")
        salarys = self.soup.select("table.newlist > tr > td.zwyx")
        locations = self.soup.select("table.newlist > tr > td.gzdd")
        times = self.soup.select("table.newlist > tr > td.gxsj > span")
        for name, salary, location, time in zip(job_name, salarys, locations, times):
            self.job_data = {
                'name': name.get_text(),
                'salary': salary.get_text(),
                'location': location.get_text(),
                'time': time.get_text(),
                'job-page':name.get('href')
            }
            #print(data)
            print('''   ------- %s
            工资: %s
            地区: %s
            状态: %s
            详情: %s
            '''% (self.job_data['name'],self.job_data['salary'],self.job_data['location'],self.job_data['time'],self.job_data['job-page']))

if __name__ == '__main__':
    t = Job_Search(url)
    #print(t.pages)
    p = Pool(5)
    for i in range(1,int(t.pages)):
        p.apply_async(t.get_job(),args=(i,))
    p.close()
    p.join()