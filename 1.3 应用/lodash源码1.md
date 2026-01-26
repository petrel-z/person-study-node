Array.fill(array, value, start, end)
array: 要填充的数组
value: 要填充的值
start: 填充的起始索引，默认为0
end: 填充的结束索引，默认为array.length

```js
   function fill(array, value, start, end) {
      var length = array == null ? 0 : array.length;
      if (!length) {
        return [];
      }
      if (start && typeof start != 'number' && isIterateeCall(array, value, start)) {
        start = 0;
        end = length;
      }
      return baseFill(array, value, start, end);
    }
```
```js
var length = array == null ? 0 : array.length;
// 当array数组是null的情况下，length等于0，否则length等于array数组的长度
if (!length) {
    return [];
}
// 如果length为0，直接return

```



```js
 function fill(array, value, start, end) {
      var length = array == null ? 0 : array.length;
      if (!length) {
        return [];
      }
      if (start && typeof start != 'number' && isIterateeCall(array, value, start)) {
        start = 0;
        end = length;
      }
      return baseFill(array, value, start, end);
}

// 是否是可迭代对象
function isIterateeCall(value, index, object) {
      if (!isObject(object)) {
        return false;
      }
      var type = typeof index;
      if (type == 'number'
            ? (isArrayLike(object) && isIndex(index, object.length))
            : (type == 'string' && index in object)
          ) {
        return eq(object[index], value);
      }
      return false;
}

// 是否是对象
function isObject(value) {
      var type = typeof value;
      return value != null && (type == 'object' || type == 'function');
}

// 是否是类数组
 function isArrayLike(value) {
      return value != null && isLength(value.length) && !isFunction(value);
}

// 是否是有效长度
// 条件1：类型必须是数字
// 条件2：必须是正数（大于-1，即≥0）
// 条件3：必须是整数（不能是小数）
// 条件4：不能超过最大安全整数
function isLength(value) {
      return typeof value == 'number' &&
        value > -1 && value % 1 == 0 && value <= MAX_SAFE_INTEGER;
}

// 更精准、更兼容地判断一个值是否为函数
function isFunction(value) {
      if (!isObject(value)) {
        return false;
      }
      // The use of `Object#toString` avoids issues with the `typeof` operator
      // in Safari 9 which returns 'object' for typed arrays and other constructors.
      var tag = baseGetTag(value);
      return tag == funcTag || tag == genTag || tag == asyncTag || tag == proxyTag;
    }

// 填充数组
function baseFill(array, value, start, end) {
      var length = array.length;

      start = toInteger(start);
      if (start < 0) {
        start = -start > length ? 0 : (length + start);
      }
      end = (end === undefined || end > length) ? length : toInteger(end);
      if (end < 0) {
        end += length;
      }
      end = start > end ? 0 : toLength(end);
      while (start < end) {
        array[start++] = value;
      }
      return array;
}
```