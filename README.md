# python
学习
# import pprint

#
# import requests
#
# keyword = input("输入关键字")
# # url = f'https://www.sogou.com/web?query={keyword}'
# # header = {'User-Agent': 'Mozilla/5.0 (iPad; CPU OS 18_5 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/18.5 Mobile/15E148 Safari/604.1'}
# #
# # response = requests.get(url=url,headers=header)
# #
# # print(response.text)
# # print(response.request.headers)
# #
# # with open(keyword + '.html', 'w', encoding='utf-8') as f:
# #     f.write(response.text)
#
#
# query_url = 'https://fanyi.sogou.com/reventondc/suggV3'
#
# header = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36 Edg/142.0.0.0'}
#
# data = {
#         'from': 'auto',
#         'to': 'zh-CHS',
#         'client': 'web',
#         'text': keyword,
#         'uuid': 'fc463d00-ff4b-41ec-a76d-48bf9d4792c9',
#         'pid': 'sogou-dict-vr',
#         'addSugg': 'on'
# }
#
# res = requests.post(url=query_url,headers=header,data=data)
# result = res.json()['sugg'][0]['v']
# pprint.pprint(result)
import json
import pprint

import requests


def get_params():
    input_params = [
        input("请输入查询的类型,默认为全部: ").strip(),
        input("请输入查询的地区,默认为全部: ").strip(),
        input("请输入查询的年份,默认为全部: ").strip(),
        input("请输入要查询的页数: ").strip(),
        input("请输入你想要的评分1-10,默认为7：").strip()
    ]

    tags = ','.join(filter(None, input_params[:3]))
    try:
        pages = int(input_params[3]) * 20 if input_params[3] and int(input_params[3]) > 0 else 20
    except ValueError:
        pages = 20

    try:
        expected_rate = float(input_params[4]) if input_params[4] and 1 <= float(input_params[4]) <= 10 else 7.0
    except ValueError:
        expected_rate = 7.0

    selected_categories = {
        "类型":input_params[0],
        "地区":input_params[1]
    }
    selected_categories = json.dumps(selected_categories, ensure_ascii=False)


    return tags, selected_categories, pages, expected_rate

tags, selected_categories, pages, expected_rate = get_params()

for pages in range(0,pages,20):
    url = 'https://m.douban.com/rexxar/api/v2/movie/recommend'

    parameters = {
            'refresh': 0,
            'start': pages,
            'count': 20,
            'selected_categories': selected_categories,
            'uncollect': 'false',
            'score_range': '0,10',
            'tags': tags
        }
    # print(parameters,expected_rate)

    header = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36 Edg/142.0.0.0', 'Referer': 'https://m.douban.com/movie/'}

    response = requests.get(url=url,headers=header,params=parameters).json()['items']
    for info in response:
        rating = info.get('rating', {})
        rate = rating.get('value')
        title = info.get('title')
        if rate is not None:
            if float(rate) >= expected_rate:
                print(title, rate)
        else:
            print(f"跳过 '{title}' - 无评分信息")

# pprint.pprint(response)
