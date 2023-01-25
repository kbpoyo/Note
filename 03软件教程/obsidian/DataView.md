# DataView

**定义：**
Obsidian资料库的查询工具/插件
- 查询对象：Obsidian数据库
- 查询依据：YAML数据/Meatainfo

**场景：**
- 什么时候使用[[Obsidian基础#2 Obsidian中的搜索|Obsidian的搜索]]
	- 条件单一
	- 无需保存结果
- 什么时候使用DataView查询
	- 条件复杂(条件到达三个及以上)
	- 需要保存结果

**格式一**
>```dataview
>list（以列表的方式展现）
>from "02软件教程"  （引号中为搜索的文件夹名）
>```

> [!example]+ 查询“02软件教程”文件夹下的文章并以列表形式展现
> ```dataview
> list
> from "02软件教程"
> ```

**格式二**
>```dataview
>table file.size as 文件大小 (以表格的方式展现)
>from #Markdown (查询Markdown标签)
>where (支持条件查询，条件元素为文章的YMAL数据)
>sort（支持排序，**asc** ，升序；**dsc** ，降序）
>```

> [!example]+ 查询带有Markdown标签的文章并以表格的形式展现
> ```dataview
> table file.size as 文件大小
> from  #Markdown 
> ```



