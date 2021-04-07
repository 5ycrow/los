---
layout: post
title: element-ui中el-table相关操作
date: 2021-04-07
Author: Los
tags: [vue,elementui]
comments: false
toc: true
---

## vue-element系列 内table插入超链接 a 标签用法

```vue
<el-table-column :label="$t('es.cloud_url')" min-width="15px" align="center">
   <template slot-scope="{row}">
       <el-link :href="row.cloud_url" target="_blank" class="buttonText"  type="primary" :underline="false">详情</el-link>
   </template>
</el-table-column>
```
## vue+element-ui实现表格内嵌套el-image自动绑定tableData中的图片url，并实现点击大图功能

```vue
    <el-table-column type="expand">
      <template slot-scope="props">
		  <el-form label-position="left" inline class="demo-table-expand">
			<el-form-item label="图片" style="width: 200px;">
			  <el-image
			  style="width: 200px;height: 200px;"
			  :src="props.row.picurl"
			  :preview-src-list="[props.row.picurl]"></el-image>
			</el-form-item>
		  </el-form>
        <el-form label-position="left" inline class="demo-table-expand">
          <el-form-item label="内容" style="width: 1200px;">
            <span>{{ props.row.content }}</span>
          </el-form-item>
        </el-form>
      </template>
    </el-table-column>
```