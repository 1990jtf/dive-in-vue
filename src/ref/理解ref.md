## 有reacitve了，为什么还要有ref？
在JavaScript中的`Proxy`无法提供对原始值的代理。

## 原始值
原始值是 `Boolean`、`Number`、`BigInt`、`String`、`Symbol`、`undefined` 和 `null` 等类型的值。

## 理解ref
ref的本质上一个“包裹对象”，“包裹对象”本质上与普通对象没有任何区别，因此为了区分 `ref` 与 `普通响应式对象` ，增加了一个标识，`__v_isRef`。

## ref的作用 ⭐️
1. 原始值的响应式方案；
2. 用来解决响应式丢失问题;

## ref的实现
+ ref功能的实现；
+ toRef/toRefs 功能实现（解决响应式丢失）;

```javascript
// 封装一个 ref 函数
function ref(val){
    // 在 ref 函数内部撞见包裹对象
    const wrapper = {
        value: val
    }
    // 将包裹对象对象编程响应式数据
    return reactive(wrapper)
}
```
封装一个 `ref` 函数，且包含一个特有的标识符 `__v_isRef`（方便ref与普通对象做区分）。

```javascript
// 如何区分
const refVal1 = ref(1)
const refVal2 = reacitve({value:1})
```

```javascript
function ref(val){
    const wrapper = {
        value: val
    }
    // 使用Object.defineProperty 在wrapper对象上定义一个不可枚举的属性_v_isRef，并且值为true
    Object.defineProperty(wrapper, '__v_isRef', {
        value: true
    })

    return reactive(wrapper)
}
```

## toRef & toRefs 的实现
toRef解决响应式丢失的问题。

```javascript
function toRef(obj, key){
    const wrapper = {
        get value(){
            return obj[key]
        }
        // 允许设置值
        set value(val){
            obj[key] = val
        }
    }
    Object.defineProperty(wrapper, '__v_isRef', {
        value: true
    })
}
```

如果响应式数据obj的键key非常的多，我们还需要花费很大力气来做一层转换。
为此，我们封装了`toRefs`函数，来批量地完成转换。

```javascript
function toRefs(obj){
    const ret = {}
    // 使用 for...in 遍历对象
    for (const key in obj){
        // 逐个调用 toRef 完成转换
        ret[key] = toRef(obj, key)
    }
    return ret
}
```
只需要一步操作，就可以完成一个对象的转换：

```javascript
const newObj = { ...toRefs(obj)}

// 测试
const obj = reactive({foo: 1, bar: 2})

const newObj = { ...toRefs(obj)}
console.log(newObj.foo.value) // 1
console.log(newObj.bar.value) // 2
```

## 自动脱 ref
`toRefs` 函数的确解决了响应式丢失问题，但同时也带来了新的问题。
由于 `toRefs` 会把 响应式数据的第一层属性值转换为 ref，因此必须通过 value 属性访问值，如以下代码所示：

```javascript
const obj = reactive({foo: 1, bar: 2})
obj.foo // 1
obj.bar // 2

const newObj = { ...toRefs(obj) }
// 必须使用 value 访问值
newObj.foo.value // 1
newObj.bar.value // 2
```

为了降低用户的心智负担，需要自动脱 ref 的能力。所谓自动脱 `ref` ，指的是属性的访问行为，即如果读取的属性是一个 `ref` ，则直接将 ref 对应的 `value` 属性值返回。

```javascript
// 自动脱ref的实现
function proxyRefs(target){
    // 返回代理对象
    return new Proxy(target, {
        get(target, key, receiver) {
            const value = Reflect.get(target, key, receiver)
            // 自动脱 ref实现： 如果读取的值是 ref，则返回它的 value 属性值
            return value.__v_isRef ? value.value : value
        }
    })
}

// 调用 proxyRefs 函数创建代理
const newObj = proxyRefs({ ...toRefs(obj) } )
```
实际上，在编写 `Vue.js` 组件时，组件中的 `setup` 函数所返回的数据会传递给 `proxyRefs` 函数进行处理：

```javascript
const MyComponent = {
    setup() {
        const count = ref(0)

        // 返回的这个对象会传递给 proxyRefs
        return {
            count
        }
    }
}
```
这也是为什么我们可以在模版直接访问一个ref的值，而无需通过 `value` 属性来访问。

```javascript
<p>{{ count }}</p>
```

实际上，自动脱 ref 不仅存在于上述场景。在 `Vue.js` 中，`reactive` 函数也有自动脱 ref 的能力，如下所示：

```javascript
const count = ref(0)
const obj = reactive({ count })

obj.count // 0
```

## 参考资料
1. 《Vue源码》原始值的响应式方案 P160