# WebCrawlerForShenzhenExchange
深圳互动易公司问答数据爬虫
![Demo](https://github.com/jason5306/WebCrawlerForShenzhenExchange/blob/main/DemoImg.png)

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.keys import Keys
from bs4 import BeautifulSoup
import requests
import time
import re
import pandas as pd

# 深交所公司代码
code = pd.read_excel('./code.xlsx', header = 0, dtype=str)
code_list = code['code'].tolist()


#for code in code_list:
driver = webdriver.Chrome()
# a new window for QA site
driver2 = webdriver.Chrome()
url = 'http://irm.cninfo.com.cn/ircs/index'

for code in code_list:

    print('Collecting data for company ... ' + code + '...........')

    driver.set_window_size(1000,30000)

    driver.get(url)
    table_content_translate_box = []
    #while(True): # Stop manually 

    # scoll down
    #driver.execute_script("window.scrollBy(0,8000)")
    #time.sleep(1)

    # find search box and search company code
    time.sleep(2)
    searc_box = driver.find_element_by_class_name('el-input__inner')
    searc_box.click()
    time.sleep(1)
    searc_box.send_keys(code)
    searc_box.send_keys(Keys.ENTER)
    time.sleep(1)

    # go to QA section
    tablist = driver.find_element_by_css_selector('.el-tabs__nav.is-top')
    tablist.find_element_by_id('tab-2').click()

    # find date search
    date_search_box = driver.find_element_by_class_name('el-range-input')
    date_search_box.click()
    date_search_box.clear()
    time.sleep(1)
    date_search_box.send_keys('2012-01-01')
    time.sleep(1)
    action = ActionChains(driver)
    action.move_by_offset(0,0).click().perform()


    # max page number
    page = driver.find_element_by_class_name('el-pager')
    pages = page.find_elements_by_class_name('number')
    page_number = []
    for item in pages:
        page_number.append(int(item.text))
        #print(item.text)

    max_page_number = max(page_number)
    #print(max_page_number)

    #data_ids = []

    # header
    result = []
    header = ['深市代码', '提问', '提问时间', '回复', '回复时间']
    result.append(header)

    while True:
        # current page number
        page = driver.find_element_by_class_name('el-pager')
        active_page_number = page.find_element_by_css_selector('.number.active').text
        print("Current page:" + str(max_page_number) + '/' + str(active_page_number))


        # find QA of this page 
        search_content_box = driver.find_elements_by_class_name('search-content-box')
        time.sleep(1)
        for item in search_content_box:
            temp_item1 = item.find_element_by_css_selector('.item-question.left-content.table-list.pd-r-20')
            temp_item2 = temp_item1.find_element_by_css_selector('.table-content.div-translate')
            data_id = temp_item2.get_attribute('data-id')
            print(data_id)
            #data_ids.append(data_id)
            time.sleep(1)

            # get QA for each data id
            url_QA = 'http://irm.cninfo.com.cn/ircs/question/questionDetail?questionId=' + data_id
            driver2.get(url_QA)
            time.sleep(3)
            # have Q and A
            Q = driver2.find_element_by_css_selector('.question-detail.translate-box.pd-30').text
            time.sleep(1)
            print(Q)
            Qtime = driver2.find_element_by_class_name('question-time').text.split('·')[0]
            time.sleep(1)
            print(Qtime)
            try:
                A = driver2.find_element_by_css_selector('.left-content.no-data').text
                print(A)
                Atime = '暂无回复'
                print(Atime)
            except Exception:

                A = driver2.find_element_by_css_selector('.reply-content.translate-box').text
                print(A)

                Atime = driver2.find_element_by_css_selector('.question-time.hidden-xs-only').text.split('：')[1]
                print(Atime)
            print('________________')
            time.sleep(2)
            result.append([code, Q, Qtime, A, Atime])


        print('Page finished')

        # next page
        next_btn = driver.find_element_by_class_name('btn-next')
        next_btn.find_element_by_css_selector('.el-icon.el-icon-arrow-right').click()
        time.sleep(1)
        if int(active_page_number) == max_page_number:
            print("All pages done")
            break
    # export to csv        
        df = pd.DataFrame(result)
        df.to_csv('./'+code+'.csv', index = False, header = 0)
        
```
