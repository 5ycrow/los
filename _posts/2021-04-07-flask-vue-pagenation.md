---
layout: post
title: vue+flask分页问题
date: 2021-04-07
Author: Los
tags: [Vue,Flask]
comments: false
---

## 处理大表数据分页问题（Flask+vue）

小表数据分页交给前端切片：

```vue
  <el-table
    :data="tableData.slice((currentPage-1)*pageSize,currentPage*pageSize)"
    style="width: 100%">
```
大表数据交给后端切片：
在查询数据时使用.limit(),.offset()
limit：限制本次查询条数
offset：从头开始隐藏查询的条数

前端js传值
offset:(this.currentPage-1)*this.pageSize,limit:this.pageSize
后端利用
zhihu=Zhihu.query.offset(offset).limit(limit).all()
即 实现切片分页查询
同时 注意 后端 要将该表数据条目总数传给前端，让前端进行预分页
每次页面条数变化，或者页面码数变化执行一次查询操作

注意
```python
        #total=Cucnews.query.count()
        total = Cucnews.query.all()
        print(len(total))
```
第一条语句会获取具体表内容，对于大表查询会拖慢速度


前端axios封装get方法传参的参数为params，post为data即：
```js
  getNews (params) {
    return get('/cucnews/getnews',params)
  },
  searchNews (data) {
    return post('/cucnews/getnews',data)
  }
```
## get方法传接参：

```js
api.getZhihu({offset:(this.currentPage-1)*this.pageSize,limit:this.pageSize}).then(res => {

}
```
```python
        offset = request.args.get("offset")
        limit = request.args.get("limit")
```
## post方法传接json：

```js
api.searchZhihu(JSON.stringify({wanted: this.wanted})).then(res => {

}
```
```python
        params=request.get_json()
        # print(params)
        wanted = params['wanted']
        offset=params['offset']
        limit=params['limit']
```



