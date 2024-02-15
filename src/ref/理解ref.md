## 有reacitve了，为什么还要有ref？
在JavaScript中的`Proxy`无法提供对原始值的代理。

## 原始值
原始值：`Boolean`、`Number`、`BigInt`、`String`、`Symbol`、`undefined` 和 `null` 等类型的值。

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
封装一个 `ref` 函数，且包含一个特有的标识符 `__v_isRef`（方便与普通对象做区分）。

```javascript
function ref(val){
    const wrapper = {
        value: val
    }
    // 使用Object.defineProperty 在wrapper对象上定义一个不可枚举的属性
    // __v_isRef，并且值为true
    Object.defineProperty(wrapper, '__v_isRef', {
        value: true
    })

    return reactive(wrapper)
}
```