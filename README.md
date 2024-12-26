# get_Top250

#python爬取豆瓣电影top250并可视化基本思路




import os
import requests  # 网络请求
from lxml import etree  # 数据解析
import pandas as pd  # 数据分析
from matplotlib import pyplot as plt#统计图绘制

# 中文显示
plt.rcParams['font.sans-serif'] = ['SimHei']
plt.rcParams['axes.unicode_minus'] = False

# 全局变量定义
excel = "豆瓣电影top250.xlsx"
csv = "豆瓣电影top250.csv"

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3'
}


def get_first_text(list):
    try:
        return list[0].strip()
    except:
        return ""  
# 转为字符串输出

def getData():
    df = pd.DataFrame(columns=["序号", "标题", "链接", "导演", "评分", "评价人数", "简介"])

    urls = ['https://movie.douban.com/top250?start={}&filter='.format(str(i * 25)) for i in range(10)]
    count = 1
    for url in urls:
        res = requests.get(url=url, headers=headers)
        html = etree.HTML(res.text)
        lis = html.xpath('//*[@id="content"]/div/div[1]/ol/li')
        # 解析数据
        for li in lis:
            title = get_first_text(li.xpath('./div/div[2]/div[1]/a/span[1]/text()'))  # 电影标题
            src = get_first_text(li.xpath('./div/div[2]/div[1]/a/@href'))  # 电影链接
            dictor = get_first_text(li.xpath('./div/div[2]/div[2]/p[1]/text()'))  # 导演
            score = get_first_text(li.xpath('./div/div[2]/div[2]/div/span[2]/text()'))  # 评分
            comment = get_first_text(li.xpath('./div/div[2]/div[2]/div/span[4]/text()'))  # 评价人数
            summary = get_first_text(li.xpath('./div/div[2]/div[2]/p[2]/span/text()'))  # 电影简介
            print(count, title, src, dictor, score, comment, summary)  # 输出

            df.loc[len(df.index)] = [count, title, src, dictor, score, comment, summary]

            count += 1

    df.to_excel(excel, sheet_name="豆瓣电影top250", na_rep="")
    df.to_csv(csv)
    print("已生成")


def dataV():
    """
    进行电影数据的简单可视化
    """
    df = pd.read_csv(filepath_or_buffer=csv)
    rating_counts = df['评分'].value_counts().sort_index()
    fig, ax = plt.subplots(figsize=(10, 6))
    ax.pie(rating_counts, labels=rating_counts.index, autopct='%1.1f%%', startangle=90)
    ax.axis('equal')  # Equal aspect ratio ensures that pie is drawn as a circle.
    ax.set_title('豆瓣电影Top 250评分分布')
    os.makedirs("images", exist_ok=True)
    plt.savefig('images/豆瓣电影Top 250评分分布.png')
    plt.show()

def main():

    getData()
    dataV()


if __name__ == '__main__':
    main()
