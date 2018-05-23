**author**

[bit4](https://github.com/bit4woo)

**domain_hunter**

A Burp Suite extender that search *<u>**sub domains and similar domains**</u>* from sitemap. Some times similar domain give you suprise^_^. that's why I care about it.

**usage**

1. download this burp extender from [here](https://github.com/bit4woo/domain_hunter/releases).
2. add it to burp suite. you will see a new tab named “Domain Hunter”, if  no error encountered. 
3. visit your target website(or App) with burp proxy enabled, ensure burp recorded http or https traffic of your target.
4. you can just switch to the "domain hunter" tab, input the domain that you want to search and click "search" button.
5. or you can  run "spider all" first to try to find more subdomains and similiar domains. 

**screenshot**

![screenshot](doc/domain-hunter-v0.3.png)

**change log**

2017-07-28: Add a function to crawl all known subdomains; fix some bug.

**xmind of domain collection**

![xmind](doc/xmind.png)

**Burp插件微信交流群**：

![wechat_group](doc/wechat_group.jpg)

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import json
from urllib import unquote
import re
from flask import Flask
from flask import request
app = Flask(__name__)
import requests
from flask import Response

sessid = 'im9ds0ji2hq8farodea1qr17k0'


def get_headers(sessid='im9ds0ji2hq8farodea1qr17k0'):
    headers = {
            'Host': 'www.qichacha.com',
            'User-Agent': r'Mozilla/5.0 (Windows NT 6.1; WOW64; rv:55.0) Gecko/20100101 Firefox/55.0',
            'Accept': '*/*',
            'Accept-Language': 'zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3',
            'Accept-Encoding': 'gzip, deflate',
            'Referer': 'http://www.qichacha.com/',
            'Cookie': r'UM_distinctid=16FFFF*c02;PHPSESSID=%s;acw_sc__=5afea0a206a65985896d333ccccf5f27ae174aa8;' % sessid,
            'Connection': 'keep-alive',
            'If-Modified-Since': 'Wed, 30 **********',
            'If-None-Match': '"59*******"',
            'Cache-Control': 'max-age=0',
        }
    return headers


def get_org_id_v2(keyword):
    global orgID
    s = requests.session()
    url = 'https://www.qichacha.com/gongsi_getList'
    data = {'key':keyword, 'type':'0'}
    try:
        resp = requests.post(url, headers = get_headers(sessid), data=data).content
        orgID = json.loads(resp)[0].get('KeyNo')
        return orgID
    except Exception,e:
            print e


def get_relation(orgID):
    s = requests.session()
    url = "https://www.qichacha.com/company_muhouAction?keyNo=%s" % orgID
    print url
    resp = s.get(url, headers=get_headers(sessid)).content
    return resp

def get_cmsmap(orgID):
    s = requests.session()
    url = "https://www.qichacha.com/cms_map?keyNo=%s&upstreamCount=4&downstreamCount=4" % orgID
    print url
    resp = s.get(url, headers=get_headers(sessid)).content
    return resp


@app.route('/')
def hello_world():
    return 'helo!'

@app.route('/api/<company>')
def org_relation(company):
    orgID = get_org_id_v2(company)
    return get_relation(orgID)
    # return 'helo!'

@app.route('/api/cmsmap/<company>')
def cmsmap(company):
    orgID = get_org_id_v2(company)
    return get_cmsmap(orgID)


@app.route('/update', methods=['GET', 'POST'])
def update():
    global sessid
    if request.method == 'POST':
        sessid = request.form.get('sessid')
        return 'ok!'



if __name__ == '__main__':
    orgID = ''
    app.run("127.0.0.1", 8000)
```
