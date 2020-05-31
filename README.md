
import requests
import csv
from bs4 import BeautifulSoup


URL = 'https://mestam.info/ru/rostov-na-donu/administraciya-goroda-gorodskogo-okruga'
HEADERS = {'User-Agent' : 'Mozilla/5.0 (Windows NT 6.3; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.61 Safari/537.36', 'Accept': '*/*'}
HOST ='https://mestam.info'
FILE ='documents.csv'

def get_html(url, params=None):
    r = requests.get(url, headers=HEADERS, params=params)
    return r

def get_content(html):
    soup = BeautifulSoup(html, 'html.parser')
    items = soup.find_all('div', class_='ol-item')


    documents=[]

    for item in items:
             documents.append({
        'title': item.find('h3', class_='oli-title').get_text(strip=True),
        'link': HOST + item.find('a').get('href'),
        'address' : item.find('i',class_='fa fa-map-signs size-15').find_next('span').get_text(strip=True),
        'phone': item.find('i', class_='fa fa-phone size-15').find_next('span').get_text(strip=True),
       
        'administration': item.find('li', class_='oli-row attributes layout__container flex-children__center').get_text(strip=True)
        })
    return documents

def save_file(items, path):
    with open(path, 'w', newline='') as file:
        wrater = csv.writer(file, delimiter=';')
        wrater.writerow(['Название','Ссылка','Адрес','Телефон','Ближайшая остановка','Администрация'])
        for item in items:
            wrater.writerow([item ['title'],item ['link'],item ['address'],item ['phone'],item ['bus_stop'],
                             item ['administration'],])

def parse():

    html = get_html(URL)
    if html.status_code == 200:
        documents = []
            
        documents.extend(get_content(html.text))
        print(documents)
        save_file(documents, FILE)
       
    else:
        print('Error')
parse()
