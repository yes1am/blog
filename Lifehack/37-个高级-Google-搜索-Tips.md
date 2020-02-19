[英文原版](https://www.coforge.com/blog/advanced-google-search-tips)

## 1. 使用引号包裹关键字

用引号来包裹关键字，这样谷歌会进行精确的短语搜索。  

1. 使用场景  

找到关于 direct sales 的课程或者培训，它可以是任意类型的课程和培训，但是，它必须包含 "direct sales" 这个关键词  

2. 语法

"[关键词1] [关键词2]" [关键词3]  

3. 示例

["direct sales" training](https://www.google.com/search?q=%22direct+sales%22+training)  

## 2. 使用 OR 分割关键词

大写字母 OR 可以用来分割两个关键词  

1. 使用场景

我在寻找 烤制shrimp 或者 烤制scallops 的食谱  

2. 语法

[关键词1] OR [关键词2]  

3. 示例

[grilled shrimp OR scallops](https://www.google.com/search?q=grilled+shrimp+OR+scallops)

## 3. 使用 - 排除某些词

1. 使用场景

你再寻找一个**自动化营销工具**，但是你不想要**邮件自动化营销工具**，于是我们可以排除 "email" 这个词。  

2. 语法

-[要排除的关键词] [要包含的关键词]  

3. 示例

[-email marketing automation tools](https://www.google.com/search?q=-email+marketing+automation+tools)  

## 4. 在 Text 中查找

`allintext` 语法只会查找一个网站的 body text，会忽略 links, URLs, and titles。  

links: 是网页内部的链接链接  
URLS: 是当前网页自身的链接  
titles: 是网站的标题，是每一项搜索结果中的**可点击区域**，如图:  


![title 区域](https://raw.githubusercontent.com/yes1am/PicBed/master/img/20200130123445.png)

1. 使用场景

查找关于 Inbound Marketing conference in 2019 的信息  

2. 语法

allintext:[keywords]  

*keywords 可以是多个单词*

3. 示例

[allintext:inbound marketing conferences 2019](https://www.google.com/search?q=allintext%3Ainbound+marketing+conferences+2019)

## 5. 在 Text + Title + URL等，中查找

`intext` 会在页面的 body text，page title，URL 中进行查找  

1. 使用场景

查找一片关于 the health benefits of running 的文章

2. 语法

intext:[keywords]  

3. 示例

[intext:health benefits running](https://www.google.com/search?q=intext%3Ahealth+benefits+running)  

注意：Google 会在 body text, page title 和 URL 中搜索 "health benefits running"

## 6. 在 Title 中搜索(一个单词)

1. 使用场景

查找一个关于 marketing 的博客，博客标题需要有 inbound 这个单词  

2. 语法

[keywords 1] intitle:[keywords 2]  

3. 示例

[marketing blogs intitle:inbound](https://www.google.com.hk/search?q=marketing+blogs+intitle%3Ainbound)  

## 7. 在 Title 中搜索 (多个单词)

1. 使用场景

搜索 best Thai restaurants in Chicago  

2. 语法

allintitle:[keywords]

3. 示例

[allintitle:bestthai food in chicago](https://www.google.com/search?q=allintitle%3Abest+thai+food+in+chicago)  

## 8. 在 URL 中搜索

1. 使用场景

搜索一片关于 SEO competitor research 的文章，限制搜索结果的 URL 必须包含**关键词**

2. 语法

allinURL:[keywords]

3. 示例

[allinURL:seo competitor research](https://www.google.com/search?q=allinURL%3Aseo+competitor+research)

## 9. 在 WebSite 中搜索

1. 使用场景

在 New York Times 网站中搜索 nyse  

2. 语法

site:[website URL] [keywords]

3. 示例

[site:nytimes.com nyse](https://www.google.com/search?q=site%3Anytimes.com+nyse&oq=site%3Anytimes.com+nyse)  

## 10. 搜索近义词

1. 使用场景

搜索 direct marketing "strategies"(策略) ，其中 "strategies" 可以是 tips 或者 best practices 等这些近义词

2. 语法

"[keywords]" ~[近义词]

3. 示例

["direct marketing" ~strategies](https://www.google.com/search?q=%22direct+marketing%22+~strategies)

## 11. 搜索相似内容的网站

1. 使用场景

像找一个和 HubSpot 类似的**入站营销软件**  

2. 语法

related:[你所熟悉的网站 URL]

3. 示例

[related:hubspot.com](https://www.google.com/search?q=related%3Ahubspot.com)  

## 12. 搜索某个事物的定义

不用查字典，就能知道某个事物的定义。

1. 使用场景

想找到 marketing 的定义

2. 语法

define:[keyword]

3. 示例

[define:marketing](https://www.google.com/search?q=define%3Amarketing)

## 13. 搜索价格

Google 可以根据价格来搜索物品

1. 使用场景

新款的 Amazon echo plus 大概要 $149, 我想看看有没有更便宜一些的

2. 语法

[keywords] $[number]

3. 示例  

想找找是否有 99.99 的 amazon echo plus  

[amazon echo plus $99.99](https://www.google.com/search?q=amazon+echo+plus+%2499.99)  

## 14. 基于地理位置搜索

为了找到一个特定地点的 news 或者 information，输入搜索关键词，紧接着跟上 `location:` 命令  

1. 使用场景

你听说  Avengers(复仇者联盟) 会在 Georgia 上映，你想要找到在 Georgia 这个地方的相关信息.

2. 语法

avengers location:georgia

3. 示例  

[avengers location:georgia](https://www.google.com/search?q=avengers+location%3Ageorgia)  

## 15. Google 搜索通配符(当你忘了某些单词的时候)

1. 使用场景

你只记得 "Humpty Dumpty" 中的一些单词，你想要搜索这首歌

2. 语法

[keywords 1] * [keywords 2]

3. 示例  

[Humpty Dumpty * a wall](https://www.google.com/search?q=Humpty+Dumpty+*+a+wall)  

## 16. 翻译

1. 使用场景

想将 the little baby is playing 翻译成 Spanish

2. 语法

translate [keyword or sentence] to [language]

3. 示例  

[translate "the little baby is playing" to Spanish](https://www.google.com/search?q=translate+the+little+baby+is+playing+to+spanish)  

## 17. 搜索文件

搜索 PPT 或者 PDF 文件

1. 使用场景

想要搜索关于 marathon training plans 的 PPT

2. 语法

[keywords] filetype:[file type extension]

3. 示例  

[marathon training plan filetype:ppt](https://www.google.com/search?q=marathon+training+plan+filetype%3Appt)

## 18. 区域码( area code )搜索

1. 使用场景1

想找到区域码 212 对应的地址

2. 语法1

area code [3-digit area code]

3. 示例1 

[area code 212](https://www.google.com/search?q=area+code+212)

4. 使用场景2

想找到 New York City 的区域码

5. 语法2

area code [location]

6. 示例1 

[area code New York City](https://www.google.com/search?q=area+code+New+York+City)

## 19. 测量单位换算

1. 使用场景

想将 32 miles (美国的距离测量单位) 转换为 kilometers (公用的距离测量单位).

2. 语法

convert [data value + unit of measure] to [like unit of measure]

3. 示例 

[convert 32 miles to km](https://www.google.com/search?q=convert+32+miles+to+km)  

## 20. 邮政编码查询

1. 使用场景

想要找到白宫的邮政编码

2. 语法

[street number] + [street name] + [city] + [state]

3. 示例

[1600 Pennsylvania Ave, Washington, DC](https://www.google.com/search?q=1600+Pennsylvania+Ave%2C+Washington%2C+DC)  

## 21. 查询股票信息

输入一个公司的股票代码，来搜索最新的股票信息。  

如果你不知道公司的股票代码，使用第二种语法。  

1. 使用场景

查询 Microsoft 的股票信息

2. 语法1

[stock ticker symbol]

3. 语法2

[company name] stock

4. 示例

[msft](https://www.google.com/search?q=msft)  

## 22. 计算器

operator 支持: + - * /

1. 使用场景

想计算 20 * 25

2. 语法

[number] [operator] [number]

3. 示例 

[20*25](https://www.google.com/search?q=20*25)  

## 23. 小费计算器

根据总消费金额，计算小费

[tip calculator](https://www.google.com/search?q=tip+calculator)

## 24. 以数字区间来搜索

1. 使用场景

想搜索 Star Wars(星球大战) 从 1977年到1983年的信息

2. 语法

[keywords] [first date]..[second date]

3. 示例 

[star wars 1977..1983](https://www.google.com/search?q=star+wars+1977..1983)  

## 25. 秒表功能

[stopwatch](https://www.google.com/search?q=stopwatch)

## 26. 定时器

1. 使用场景

你需要一个 ten-minute 的定时器

2. 语法

[length of time] timer

3. 示例 

[10 minute timer](https://www.google.com/search?q=10+minute+timer)  

## 27. 查询日出和日落时间

找到特定地点，某一天的日出日落时间

1. 语法1

sunrise 然后回车，谷歌会自动定位你的地理位置，然后得到日出时间

2. 语法2

sunrise New Haven, CT

3. 语法3

sunset los angeles July 4, 2019

4. 示例

[sunset madrid spain](https://www.google.com/search?q=sunset+madrid+spain)

## 28. 天气数据

1. 使用场景

想查询 Seattle, Washington 的天气怎么样

2. 语法1

weather [location]  

3. 语法2

[location] weather

4. 示例

[weather seattle wa](https://www.google.com/search?q=weather+seattle+wa)

## 29. 航班动态

1. 使用场景

查询 Delta flight 101 的航班状态

2. 语法1

[airline name] [fight number]

3. 语法2

[airline abbreviation] [fight number]

4. 示例

[delta 101](https://www.google.com/search?q=delta+101&oq=delta+101)

## 30. 运动会得分

1. 使用场景

查询 Champions League 比赛, Manchester United 和 PSG 的比赛得分情况  

2. 语法

[team 1] [keywords] [team 2]

3. 示例

[manchester united vs psg](https://www.google.com/search?q=manchester+united+vs+psg)  

## 31. 营养学知识

通过食品名称，查询脂肪含量，卡路里，营养素等信息

1. 使用场景

查询 grilled chicken salad 的营养信息

2. 语法

[food item] nutrition

3. 示例

[grilled chicken salad nutrition](https://www.google.com/search?q=grilled+chicken+salad+nutrition)  

## 32. 图片搜索

1. 使用场景

查找 sales funnel 的 PNG 格式图片

2. 语法

[keyword] image type

3. 示例

[sales funnel png](https://www.google.com/search?q=sales+funnel+png)  

## 33. 电影搜索

1. 使用场景

查询电影 Avengers Movie 的相关信息

2. 语法

movie:[new movie name]

3. 示例

[movie:avengers](https://www.google.com/search?q=movie%3Aavengers)

## 34. 让页面翻滚(娱乐)

[do a barrel roll](https://www.google.com/search?q=do+a+barrel+roll)

## 35. 抛硬币(娱乐)

[Flip A Coin](https://www.google.com/search?q=flip+a+coin)

## 36. 倾斜的页面(娱乐)

[askew](https://www.google.com/search?q=askew)

## 37. 游戏(娱乐)

[google pacman](https://www.google.com/search?q=google+pacman)

[Atari Breakout](https://www.google.com/search?q=atari+breakout&tbm=isch)

[zerg rush](https://www.google.com/search?q=zerg+rush)