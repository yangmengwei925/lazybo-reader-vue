# vue翻页小demo

[vue翻页小demo](https://github.com/lazybo-code/lazybo-reader-book)

克隆并运行
```base
git clone https://github.com/lazybo-code/lazybo-reader-book.git
yarn install
yarn serve
```

# 标签参数
|参数|说明|类型|可选值|默认值|
|-------|-------|-------|-------|-------|
|title|标题|string|—|—|
|content|内容 内容换行作为分割|string|—|—|
|clearance|左右间隔|number|—|16|
|fontSize|字体大小 rem|number|—|1.125|
|className|新增一个class|string|—|—|
|animationSeconds|切换时长 单位(秒)|number|—|0.3|

# 事件
|事件名称|说明|回调参数|
|-------|-------|-------|
|onToEnd|切换页面,在到头后提示，即将到头提示，到尾部提示，即将到尾部提示|type: 0: 到头,1: 即将到头,2: 到尾部,3: 将到尾部|

# ref方法
|方法|说明|
|-------|-------|
|previousPage|上一页|
|nextPage|下一页|
|calculateTotalPage|更新页面总数|

# slot
|方法|说明|数据|
|-------|-------|-------|
|cover-top|左上角 的标题内容|page: {total: number;current: number;};title: string;|
|cover-bottom-left|左下角的分页内容|page: {total: number;current: number;};title: string;|
|cover-bottom-right|右下角的内容|page: {total: number;current: number;};title: string;|

# 基础使用方法
```vue
<template>
  <lazybo-reader-book :title="title" :content="content"/>
</template>
```
