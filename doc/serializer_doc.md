## serializer 外置API说明
#### 序列化器继承于Field类 也就说明序列化器可以当做另一个序列化器的字段使用 
```
# BaseSerializer __init__
:param read_only: 只序列化
:param write_only: 只反序列化
:param required: 反序列化时必须存在此值
:param allow_null: 反序列化时可以为 None, ''
:param default: 默认值 可用于序列化和反序列化
:param source: 序列化时值的来源
:param validators: 反序列化时数据需要通过的验证
:param error_messages: 出现错误时的自定义描述
:param label: 字段标题
:param description: 字段描述
:param instance: 序列化的 Model 实例 不验证
:param data: 反序列化的数据 须经过验证 在内部会转化为 initial_data 
:param partial: 不验证未传入的值 require = True 也不验证
:param context: 实例生命周期内的上下文 通常用于 view -> seriallizer
:param many: 是否为多行 格式为 List[Dict]
```

### 常用方法

> [异步] `obj.external_to_internal(value)`

&emsp;&emsp;对传入的数据进行序列化转换并返回<br>
&emsp;&emsp;通常用于类型统一  int -> string<br>

> [异步] `obj.internal_to_external(value)`

&emsp;&emsp;对数据进行反序列化转换并返回<br>
&emsp;&emsp;通常用于类型统一  Decimal -> string<br>

> [异步] `obj.validate(attr)`

&emsp;&emsp;用于对内置验证器验证完成的数据进行二次自定义验证，必须返回 `attr` <br>

> [异步] `obj.run_validation(attr)`

&emsp;&emsp;执行所有字段的内置验证器 <br>
&emsp;&emsp;返回验证后的 `attr` <br>

> [异步] `obj.update(instance, validated_data)`

&emsp;&emsp;执行数据库的更新操作 validated_data 来自 obj.validate <br>
&emsp;&emsp;返回模型对象 `instance` <br>

> [异步] `obj.create(validated_data)`

&emsp;&emsp;执行数据库的创建操作 validated_data 来自 obj.validate <br>
&emsp;&emsp;返回创建后的模型对象 `instance` <br>

> [异步] `obj.save(**kwargs)`

&emsp;&emsp;在其中判断 调用 `create` 或 `update` <br>
&emsp;&emsp;返回模型对象 `instance` <br>

> [异步] `obj.before_update(validated_data, instance)`

&emsp;&emsp;更新前的操作<br>
&emsp;&emsp;返回 `validated_data`, `instance` <br>

> [异步] `obj.before_create(validated_data, instance)`

&emsp;&emsp;创建前的操作<br>
&emsp;&emsp;返回 `validated_data`, `instance` <br>

> [异步] `obj.is_valid()`

&emsp;&emsp;在其中判断 调用 `create` 或 `update` <br>
&emsp;&emsp;返回模型对象 `instance` <br>

> [异步] `obj.read_[field_name]()`

&emsp;&emsp;对序列化时对每个字段的Hook<br>

> [异步] `obj.validate_[field_name](value)`

&emsp;&emsp;对反序列化时对每个字段的验证Hook<br>

_

### 常用属性

> [异步] `obj.data`

&emsp;&emsp;对instance序列化后的数据 <br>

> [异步] `obj.errors`

&emsp;&emsp;对 initial_data 反序列化时验证错误的收集，所有验证错误都存在于此  <br>

> [异步] `obj.validated_data`

&emsp;&emsp;验证通过的数据<br>


## 验证流程

is_valid() - > run_validation()
             调用父级的 run_validation()
                validate_empty_values() 进行空值验证
                external_to_internal() 执行自定义的转换
                    调用 validate_[field_name]()
                run_validators() 执行自带的验证器
             将会返回验证后的值
             验证后的数据会进入 validate() 并返回
             验证后的值将会在 is_valid 中 赋值给 self._validated_data                       
             
    async def run_validation(self, data):
        """执行验证"""
        (is_empty_value, data) = self.validate_empty_values(data)
        if is_empty_value:
            return data
        value = await self.external_to_internal(data)
        self.run_validators(value)
        return value