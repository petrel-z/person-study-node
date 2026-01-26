在 Vue 中，`watch` 并不会自动监听所有 `ref` 或 `reactive` 对象的变化。它需要明确指定要监听的目标。以下是一些关键点：

### 1. **监听 `ref` 的值**

- 如果监听的是一个 `ref`，需要通过函数返回其 `.value`，例如：
    
    watch(() => someRef.value, (newValue, oldValue) => {
    
      console.log('值变化:', newValue, oldValue);
    
    });
    

### 2. **监听 `reactive` 对象**

- 如果监听的是一个 `reactive` 对象，默认会监听整个对象的变化（包括属性的增删改）。
    
    const state = reactive({ count: 0, name: 'Vue' });
    
    watch(() => state, (newState, oldState) => {
    
      console.log('对象变化:', newState, oldState);
    
    }, { deep: true }); // 需要设置 deep 为 true
    

### 3. **监听 `reactive` 对象的某个属性**

- 如果只想监听 `reactive` 对象的某个属性，可以通过函数返回该属性：
    
    watch(() => state.count, (newValue, oldValue) => {
    
      console.log('count 变化:', newValue, oldValue);
    
    });
    

### 4. **深度监听**

- 对于嵌套的对象或数组，默认不会深度监听，需要通过 `deep: true` 选项启用深度监听：
    
    watch(() => state, (newState, oldState) => {
    
      console.log('深度监听:', newState);
    
    }, { deep: true });
    

### 5. **监听多个数据源**

- 可以通过数组的形式监听多个 `ref` 或 `reactive` 的值：
    
    watch([() => someRef.value, () => state.count], ([newRefValue, newCount], [oldRefValue, oldCount]) => {
    
      console.log('多个值变化:', newRefValue, newCount);
    
    });
    

### 总结

- `watch` 不会自动监听所有 `ref` 或 `reactive`，需要明确指定监听的目标。
- 对于 `reactive` 对象，默认监听整个对象，但需要设置 `deep: true` 才能深度监听嵌套属性。
- 对于 `ref`，需要通过函数返回其 `.value`。