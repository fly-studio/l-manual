>  以下方法来基于select2.js、laravel.select.js

在后台中，经常需要在A功能里调取B的一个select选择列表，比如在用户里调取用户组，常规做法是在UserController中读取Role的所有数据集，然后在view中循环打印出Role的所有数据。
这样做的后果是在新建、编辑页面的Controller都需要去读取Role的数据。
在L+中，您无需如此。为**降低耦合**，select一律使用ajax获取到数据，并且可以使用搜索功能，使用方法也非常简单。如下：

前提是对应的Controller都有data方法(L+后台均有此方法)


# SELECT 列表
页面ready之后会自动去请求数据，数据不做筛选

## 模拟结构
```
<select value="value">
	<option value="data-id">data-text</option>
</select>

```
## 参数
- data-model="admin/role"
	这是一个重要的参数，表示去请求哪个controller下的数据集，比如：`http://.../admin/role/data/json?all=true` 下的数据
	数据输出格式参照 「快速后台」-「列表」

- data-id="{{id}}"

	**默认：**`{{id}}`
	
	上文的`data-id`

- data-text="{{display_name}}({{name}})"

	上文的`data-text`

	此处是指**下拉之后列表**展示的内容，可以写html

- data-selection="{{realname}}"

	**默认：** data-text

	选中之后，显示在**文本框里面**的数据，可以和 data-text 不一样，可以写html

	如果没有此参数，则和data-text展示的数据一样

- value="<{$_data.role_id}>"

	上文的`value`，参照下文的使用方法
	
	初始化选中的值，多个值用`,`(英文逗号)分隔

	> 注意：w3c规范中，select中并无此属性，此字段仅仅用于初始化时选中（selected="selected"）
	
- data-params="{&quot;q&quot;:{&quot;something&quot;:{&quot;lk&quot;:&quot;abc&quot;}}}"
	
	此参数可以不设置
	
	附加参数，表示在请求`data-model`时附加的字段，必须为标准的JSON格式：
	
	`{"q": {"something": {"lk": "abc"}, "order": {"updated_at": "desc"}  }`
	
	`"` 引号需要使用 `&quot;` 转义

- data-placeholder=""
	
	用于select2的`placeholder`

- class中加入 select-model 则开始自动调用

	或 `$('#select').selectModel();`
	
## 效果预览
### 展开所有数据

![](/assets/Image+4[1].png)

### 选中之后（多选）

多选会表现为类似于Tag的形态，

![](/assets/Image+5[1].png)

## 使用方法
### 单选
```
<select id="role_id" name="role_id" class="form-control select-model" data-model="admin/role" data-id="{{id}}" data-text="{{display_name}}({{name}})" data-placeholder="请输入关键词..." value="<{$_data.role_id}>"></select>
```
### 多选
```
<select id="role_ids" name="role_ids[]" class="form-control select-model" data-model="admin/role" data-id="{{id}}" data-text="{{display_name}}({{name}})" data-placeholder="请输入关键词..." value="<{$_data.role_ids|default:[]|implode:','}>" multiple="multiple"></select>
```

# Suggest 联想
只有当用户输入时，才去请求数据，输入的内容作为筛选的依据
## 模拟结构
```
<select value="value">
	<option value="data-id">data-text</option>
</select>
```

## 必要参数
- data-model="admin/designer"

	这是一个重要的参数，表示去请求哪个controller下的数据集，比如：`http://.../admin/designer/data/json` 下的数据
	数据输出格式参照 「快速后台」-「列表」

- data-term="realname"

	表示搜索的字段为`realname`，即 `http://.../admin/desinger/data/json?f[realname][lk]=xxx`

- data-q="ofPinyin"

	表示筛选的方法为`ofPinyin`，即 `http://.../admin/desinger/data/json?q[ofPinyin]=xxx`
	
	与`data-term`可同时存在


- data-id="{{id}}"

	**默认：**`{{id}}`
	
	上文的`data-id`

- data-text="{{title}}({{author}})"

	上文的`data-text`

	此处是指**下拉之后列表**展示的内容，可以写html

- data-selection="{{realname}}"

	**默认：** data-text

	选中之后，显示在**文本框里面**的数据，可以和 data-text 不一样，可以写html

	如果没有此参数，则和data-text展示的数据一样

- value="<{$_data.role_id}>"

	上文的`value`，参照下文的使用方法
	
	初始化选中的值，多个值用`,`(英文逗号)分隔

	> 注意：w3c规范中，select中并无此属性，此字段仅仅用于初始化时选中（selected="selected"）

- data-params="{&quot;q&quot;:{&quot;something&quot;:{&quot;lk&quot;:&quot;abc&quot;}}}"
	
	此参数可以不设置
	
	附加参数，表示在请求`data-model`时附加的字段，必须为标准的JSON格式：
	
	`{"q": {"something": {"lk": "abc"}, "order": {"updated_at": "desc"}  }`
	
	`"` 引号需要使用 `&quot;` 转义
	
- data-placeholder=""
	
	用于select2的`placeholder`

- class中加入 suggest-model 则开始自动调用

	或 `$('#suggest').suggestModel();`
	
## 效果预览
### 模糊搜索：
比如输入了：「测」，下面读取出两条测试结果

![](/assets/Image+2[1].png)

### 选中之后(单选)
![](/assets/Image+3[1].png)

## 使用方法
### 单选
```
<select name="did" id="did" value="<{$_data.did}>" class="form-control suggest-model" data-model="admin/designer" data-text="{{realname}}({{area}})" data-term="realname"  data-placeholder="请输入真实姓名查找"></select>
```

### 多选
```
<select name="dids[]" id="dids" value="<{$_data.dids|default:[]|implode:','}>" class="form-control suggest-model" data-model="admin/designer" data-text="{{realname}}({{area}})" data-term="realname"  data-placeholder="请输入真实姓名查找" multiple="multiple"></select>
```

# Tree 树形结构
结构与 `SELECT 列表` 类似，但是会用树的样式展示数据

## 模拟结构
```
<select>
	<option value="data-id" >data-text</option>
	<option value="data-id" >│ ├ data-text</option>
	<option value="data-id" >│ └ data-text</option>
	<option value="data-id" >└ data-text</option>
</select>
```
## 必要参数
- data-model="admin/designer"

	这是一个重要的参数，表示去请求哪个controller下的数据集，比如：`http://.../admin/role/data/json?all=true&tree=true` 下的数据
	数据输出格式参照 「快速后台」-「列表」

- data-id="{{id}}"

	**默认：**`{{id}}`
	
	上文的`data-id`

- data-text="{{display_name}}({{name}})"

	上文的`data-text`

	此处是指**下拉之后列表**展示的内容，可以写html

- data-selection="{{realname}}"

	**默认：** data-text

	选中之后，显示在**文本框里面**的数据，可以和 data-text 不一样，可以写html

	如果没有此参数，则和data-text展示的数据一样

- value="<{$_data.role_id}>"

	上文的`value`，参照下文的使用方法
	
	初始化选中的值，多个值用`,`(英文逗号)分隔

	> 注意：w3c规范中，select中并无此属性，此字段仅仅用于初始化时选中（selected="selected"）

- data-params="{&quot;q&quot;:{&quot;something&quot;:{&quot;lk&quot;:&quot;abc&quot;}}}"
	
	此参数可以不设置
	
	附加参数，表示在请求`data-model`时附加的字段，必须为标准的JSON格式：
	
	`{"q": {"something": {"lk": "abc"}, "order": {"updated_at": "desc"}  }`
	
	`"` 引号需要使用 `&quot;` 转义


- data-placeholder=""
	
	用于select2的`placeholder`

- class中加入 tree-model 则开始自动调用

	或 `$('#tree-select').treeModel();`

## 效果预览 
### 展开所有数据
![](/assets/Image 2.png)
### 选中数据(多选)
![](/assets/Image 3.png)

## 使用方法
### 单选
```
<select id="role_id" name="role_id" class="form-control tree-model" data-model="admin/role" data-id="{{id}}" data-text="{{display_name}}({{name}})" data-placeholder="请输入关键词..." value="<{$_data.role_id}>"></select>
```
### 多选
```
<select id="role_ids" name="role_ids[]" class="form-control tree-model" data-model="admin/role" data-id="{{id}}" data-text="{{display_name}}({{name}})" data-placeholder="请输入关键词..." value="<{$_data.role_ids|default:[]|implode:','}>" multiple="multiple"></select>
```

# 恢复控件为原生的SELECT
```
$('.select-model').selectModel('destory');
$('.suggest-model').suggestModel('destory');
$('.tree-model').treeModel('destory');

// 其实上文都是调用的select2的方法
$('.tree-model,.suggest-model,.select-model').select('destory');
```