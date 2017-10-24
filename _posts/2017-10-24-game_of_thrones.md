---
layout: post
title: 如何用R对《权利的游戏》故事情节做情绪分析？
subtitle: 对原文python部分做了修改，统一采用R实现
bigimg: /img/path.jpg
---

> learn from:[如何用R对《权利的游戏》故事情节做情绪分析？](https://mp.weixin.qq.com/s?__biz=MzI0ODcxODk5OA==&mid=2247489031&idx=3&sn=d98b64b7ed1c171baa1c058fe10740df&chksm=e99d25fedeeaace89c67097fb6602f1115e7c4b4eaba078b21bc404c7e49f1963b4fd0b67d8d&mpshare=1&scene=1&srcid=0913IUotqk7ob0sPHpRPybm2#rd)

### 情绪分析(emotional analysis)和情感分析(sentiment analysis)
情感分析的结果一般分为正向(positive)和负向(negative)，而情绪分析包含的种类就比较多。加拿大国家研究委员会(National Research Council of Canada)官方发布的情绪词典包含了8种：

1. 愤怒(anger)
2. 期待(anticipation)
3. 厌恶(disgust)
4. 恐惧(fear)
5. 喜悦(joy)
6. 悲伤(sadness)
7. 惊讶(surprise)
8. 信任(trust)


### 准备
- 数据：[《权利的游戏》第三季第9集，"The Rains of Castamere"](https://genius.com/Game-of-thrones-the-rains-of-castamere-annotated)


### 清理
- 任务：
1. 把与剧情正文无关的内容去除
2. 将数据转换成R可以直接做情绪分析的结构化数据格式

- 数据读入

```
data<-readLines("E:/workspace_r/packages_learning/Game_of_Thrones.txt")
#head(data)
#class(data)#character
```

- 去掉开头的非剧本正文内容，去掉尾部部分，去掉分割线

```
library(stringr)
data<-str_replace(data,"\\[Opening Credits\\]","")
data<-str_replace(data,"\\[End Credits\\. No music\\]","")
data<-str_replace_all(data,"-","")
```

- 删除空行

```
data<-data[-which(data=="")]
data<-data[-which(data==" ")]
```

```
#Error in check_input(x) : 
#Input must be a character vector of any length or a list of character
#vectors, each of which has a length of 1.
data<-as.vector(data)
```

### 分析

```
#install.packages("dplyr")
#install.packages("tidytext")
#install.packages("tidyr")
#install.packages("ggplot2")


library(dplyr)
library(tidytext)
library(tidyr)
library(ggplot2)

#需要把一句句的文本拆成单词，这样才能和情绪词典里的单词做匹配，从而分析单词的情绪属性
#在R里面，可以采用Tidy Text方式来做。执行语句是unnest_token

script<-data_frame(line=seq(1,length(data)),text=data)
#class(script)#"tbl_df","tbl","data.frame" not just "data.frame" 
tidy_script<-script %>% unnest_tokens(word,text)
#原先的行号依旧被保留。可以看到每一个词来自于哪一行
```

调用加拿大国家研究委员会发布的情绪词典，在tidytext包里面内置，nrc

```
tidy_script %>% 
  inner_join(get_sentiments("nrc")) %>% 
  arrange(line) %>% 
  head(10)
#可以看到，有的词对应某一种情绪属性，有的词同时对应多种情绪属性
#nrc包里面不仅有情绪，还有情感(正向和负向)
```

综合判断每一行的不同情感分别含有几个词
```
#count():count observations by group
tidy_script %>% 
  inner_join(get_sentiments("nrc")) %>% 
  count(line,sentiment) %>% 
  arrange(line) %>% 
  head(10)
```

如果以1行为单位分析情感变化，粒度过细。鉴于整个剧本包含了几百行文字，以5行作为一个基础单位，进行分析

使用index把原先的行号处理一下，分成段落。 %/% 代表整除符号，这样0-4行就成了第一段落，5-9行成为第二段落，以此类推

```
tidy_script %>% 
  inner_join(get_sentiments("nrc")) %>% 
  mutate(index=line %/% 5) %>% 
  count(index,sentiment) %>% 
  arrange(index) %>% 
  head(10)

#与原文调整count()位置，按新分段落统计情感
```

绘制柱状图，按段落统计情感，对于不同的情绪，用不同的颜色表示
```
tidy_script %>% 
  inner_join(get_sentiments("nrc")) %>% 
  mutate(index=line %/% 5) %>% 
  count(index,sentiment) %>% 
  ggplot(aes(x=index,y=n,color=sentiment)) %>% 
  + geom_col()
```

结果看不清楚，为了区别不同情绪，调用facet_wrap函数，把不同情绪拆开，分别绘制
```
tidy_script %>% 
  inner_join(get_sentiments("nrc")) %>% 
  mutate(index=line %/% 5) %>% 
  count(index,sentiment) %>% 
  ggplot(aes(x=index,y=n,color=sentiment)) %>% 
  + geom_col() %>% 
  +facet_wrap(~sentiment,ncol = 3)
```

- 疑惑：按道理来说，每一段落的内容里，包含单词数量大致相当。结尾部分情感分析结果里面，positive和negative几乎同时上升，这就让人很不解。是这里的几行太长了，还是出了什么其他的问题呢？

数据分析的关键，就是在这种令人疑惑的地方深挖进去

看看出现最多的正向和负向情感词都有哪些

```
tidy_script %>% 
  inner_join(get_sentiments("nrc")) %>% 
  filter(sentiment=="positive") %>% 
  count(word) %>% 
  arrange(desc(n)) %>% 
  head(10)

tidy_script %>% 
  inner_join(get_sentiments("nrc")) %>% 
  filter(sentiment=="negative") %>% 
  count(word) %>% 
  arrange(desc(n)) %>% 
  head(10)
```

对比两个结果，同样的词"lord"即被当成了positive又被当成了negative

处理停用词，对于每一个具体场景，都需要使用停用词表，把那些可能干扰分析结果的词扔出去

```
tidy_script %>% 
  anti_join(stop_words) %>% 
  inner_join(get_sentiments("nrc")) %>% 
  filter(sentiment=="positive") %>% 
  count(word) %>% 
  arrange(desc(n)) %>% 
  head(10)
```

结果一样，看来停用词表里没有包含需要去除的那一堆名称

### 修订停用词表

```
stop_words#data.frame:word,lexicon
custom_stop_words<-bind_rows(stop_words,
      data.frame(word=c("stark", "mother", "father", "daughter", "brother", "rock", "ground", "lord", "guard", "shoulder", "king", "main", "grace", "gate", "horse", "eagle", "servent"),
      lexicon = c("custom")))

tidy_script %>% 
  anti_join(custom_stop_words) %>% 
  inner_join(get_sentiments("nrc")) %>% 
  filter(sentiment=="positive") %>% 
  count(word) %>% 
  arrange(desc(n)) %>% 
  head(10)

tidy_script %>% 
  anti_join(custom_stop_words) %>% 
  inner_join(get_sentiments("nrc")) %>% 
  filter(sentiment=="negative") %>% 
  count(word) %>% 
  arrange(desc(n)) %>% 
  head(10)
```

把停用词表加进去，并且用filter把情感属性删除掉了

因此分析的对象是情绪(emotion),而不是情感(sentiment)

```
tidy_script %>% 
  anti_join(custom_stop_words) %>% 
  inner_join(get_sentiments("nrc")) %>% 
  filter(sentiment!="negative" & sentiment!="positive") %>% 
  mutate(index=line %/% 5 ) %>% 
  count(index,sentiment) %>% 
  ggplot(aes(x=index,y=n,color=sentiment)) %>% 
  +geom_col() %>% 
  +facet_wrap(~sentiment,ncol=3)
```

### 收获

1. 如何对网络摘取的文本做处理，从中找出正文，并且去掉空行等内容
2. 如何利用tidytext方式来处理情感分析与情绪分析
3. 如何设置自己的停用词表
4. 如何用ggplot绘制多维度切面图形