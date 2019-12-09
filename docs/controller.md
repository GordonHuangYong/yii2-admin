关于控制说明
===========

[← 模块配置](./module.html) | [`meTables` 配置 →](./me-table.html)

[TOC]

基础控制 `jinxing\admin\controllers\Controller` 
继承自 [yii\web\Controller](http://www.yiichina.com/doc/api/2.0/yii-web-controller)
所有父类的方法，都可以使用

## 受保护属性，子类需要根据实际情况重写

|属性名称 | 类型 | 默认值 | 说明|
|:----------|:---------|:---------|:-----------|
|`$modelClass`| `string`   | `\jinxing\admin\models\Admin`  | [数据使用的 model](http://www.yiichina.com/doc/api/2.0/yii-db-activerecord)|  
|`$pk`        | `string`   | `id`       | `model` 查询使用的主键 `key`|
|`$sort`      | `string`   | `id`       | 默认排序使用的字段名称|
|`$strategy`  | `string`   | `DataTables` or `JqGrid`| 使用DataTables 显示数据还是JqGrid 使用数据，根据模板视图使用的 js 修改|
|`$uploadFromClass`|`string`| `jinxing\admin\models\forms\UploadForm`| 上传使用表单验证类(UploadForm 中需要定义上传文件字段和验证类型)|
|`$strUploadPath`|`string` | `./uploads/`| 上传文件保存文件地址|

## 请求公共的方法

|方法名称           | 说明          | 相关方法|
|:----------------|:------------|:----------------------------|
|`actionIndex()`    | 显示视图页面      |
|`actionSearch()`   | 为视图文件提供数据 |[where() Provide query conditions](#where-public-method)、[getQuery() Provide query object](#getquerywhere-protected-method)、[afterSearch() Data processing after query](#aftersearcharray-protected-method)|
|`actionCreate()`   | 创建数据         |--|
|`actionUpdate()`   | 修改数据         |--|
|`actionDelete()`   | 删除数据         |--|
|`actionDeleteAll()`| 删除多条数据      |--|
|`actionEditable()` | 行内编辑         |--|
|`actionUpload()`   | 上传文件         | [ afterUpload() File upload processing](#afteruploadfilepath-field-object-protected-method)|
|`actionExport()`   | 导出数据         | [where() Provide query conditions](#where-public-method)、[getQuery() Provide query object](#getquerywhere-protected-method)、[getExportHandleParams() Provide data export processing parameters ](#getexporthandleparams-protected-method)|

## 其他方法

### `getPk()` public method
> return `string`

获取主键名称

### `where()` public method

> return `array`

用来处理查询请求字段对应后端SQL查询表达式

```php
protected function where()
{
    return [
        // 表示前端请求查询字段 id、status、type 都使用 = 查询
        [['id', 'status', 'type'], '='],
        
        // 表示前端请求查询字段 username、email 都使用 like 查询
        [['username', 'email'], 'like'],
    ];
}
```

#### 支持配置方式
1. `key` => `value` 方式

    ```php
    protected function where()
    {
        return [
            // 简单处理
            'id' => '=',
            'username' => 'like',
         
            // 复杂处理, 使用数组
            'email' => [
               'field' => 'user.email', // 修改字段
               'func'  => 'trim',       // 使用函数处理值
               'and'   => 'like',       // 使用的连接表达式   
            ],
         
            /**
             * 复杂处理，使用匿名函数
             * 
             * @param mixed  $value  前端请求过来的值
             * @param string $column 前端对应的字段名称, 不使用的话，可以不用接收
             * @return array 需要返回一个数组
             */
            'nickname' => function ($value, $key) {
                return ['like', $key, $value];
                
                // (`nickname` like '%{$value}%' or `username` like '%{$value}%')
                // return ['or', ['like', 'nickname', $value], ['like', 'username', $value]];
            },
        ];
    }
    ```
    
2. 数组方式, 和 `key` => `value` 支持配置方式一致，只是改为数组方式，可以一个表达式对应多个字段 

`[字段, 表达式]`

```php
    protected function where()
    {
        return [
            // 简单处理
            [['id', 'type'], '=']

            // 复杂处理, 使用数组
            ['email', [
                'field' => 'user.email', // 修改字段
                'func'  => 'trim',       // 使用函数处理值
                'and'   => 'like',       // 使用的连接表达式   
            ]],
        
            // 复杂处理，使用匿名函数
            [['username', 'nickname', 'name'], function ($value, $key) {
                return ['like', $key, $value];
            }],
        ];
    }
```
#### 定义默认查询条件,指定字段`where`，必须为二维数组`array`

```php
protected function where()
{
    return [
        // 定义默认查询条件
        'where' => [['=', 'status', 1], ['=', 'type', 1]],
    ];
}
```

### `getQuery($where)` protected method
> return [yii\db\Query](http://www.yiichina.com/doc/api/2.0/yii-db-query) or [yii\db\ActiveRecord](http://www.yiichina.com/doc/api/2.0/yii-db-activerecord) 

**如果是比较复杂的查询，可以复写这个方法(比如联表查询)**

```php 
    protected function getQuery($where)
    {
        return (new Query())->from('user')->leftJoin('actrive', 'user.id=active.user_id')->where($where);
    }

```

### `afterSearch(&$array)` protected method

用来对 [getQuery ()](#getquerywhere-protected-method) 方法查询出的分页数据做进一步处理(查询使用 model 查询，其实也可以在afterFind() 方法里面处理) :

```php
    protected function afterSearch(&$array) 
    {
        foreach ($array as &$value) {
            $value['created_at']  = date('Y-m-d H:i:s', $value['created_at']);
            $value['status_name'] = $value['status'] == 10 ? 'open' : 'close'; 
        }
        
        unset($value);
    }
```

### `findOne($data = [])` protected method

> return [yii\db\ActiveRecord](http://www.yiichina.com/doc/api/2.0/yii-db-activerecord)

通过请求参数查询到对象，没有会设置错误，返回false

### `afterUpload($filePath, $field, $object)` protected method

> return `string`

上传文件之后的处理，处理成功需要返回 文件保存地址

### getExportHandleParams() protected method
> return `array`

对导出的数据做格式化处理(因为显示数据时在视图里面做的格式化处理，如果导出数据也要做格式化处理，需要定义这个方法):

```php
    protected function getExportHandleParams()
    {
        return [
            // 使用匿名函数处理，$value 为查询结果中 created_at 字段的值
            'created_at' => function ($value) {
                return date('Y-m-d H:i:s', $value);
            },
            'status' => function ($value) {
                return $value == 10 ? 'open' : 'close';
            }
        ];
    }
```

## 文件上传

[前端配置参考](./me-table.html#文件上传)

为当前控制器配置文件上传处理类

```php
public $uploadFromClass = '\backend\models\form\UploadForm';
```

在 `\backend\models\form\UploadForm` 中定义上传文件字段、验证规则、验证场景

```php

class UploadForm extend yii\base\Model
{
    /**
     * @var \yii\web\UploadFile 对应前端配制的上传文件 input name = UploadForm[file]
     */
    public $file;
    
    /**
     * @var \yii\web\UploadFile 对应前端配制的上传文件 input name = UploadForm[excel]
     */
    public $execl;
    
    /**
     * 定义验证场景
     *
     * @return array
     */
    public function scenarios()
    {
        return [
            // 目前一个字段一个场景
            'file'  => ['file'],
            'excel' => ['excel'],
        ];
    }
    
    /**
     * 定义验证规则
     *
     * @return array
     */
    public function rules()
    {
        return [
            [['file'], 'image', 'extensions' => ['png', 'jpg', 'gif', 'jpeg'], 'on' => 'file'],
            [
                ['excel'], 
                'file', 
                'extensions'               => ['xls', 'xlsx'],
                'checkExtensionByMimeType' => false
                'on'                       => 'excel',
            ]
        ];
    }
}

```

>目前使用的一个上传处理表单类`UploadForm`，所以需要一个上传文件定义一个字段、场景、验证规则；当然也可以
一个上传文件一个上传表单类，但个人觉得没有这个必须


[← 模块配置](./module.html) | [`meTables` 配置 →](./me-table.html)