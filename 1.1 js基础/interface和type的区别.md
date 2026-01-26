
## 特点
interface:
1. 可扩展（继承）
2. 可合并（同名自动合并）
3. **面向对象**（相关特性）

type：
1. 不可扩展
2. 支持联合类型
3. 计算属性

interface 被称为面向对象的原因：
1. 契约性 ：定义对象必须遵守的结构和行为
2. 继承性 ：支持 extends 关键字进行继承
3. 多态性 ：同一接口可以有不同的实现
4. 抽象性 ：隐藏实现细节，只暴露必要的接口
5. 扩展性 ：支持声明合并，便于扩展
6. 实现性 ：类可以通过 implements 实现接口

## 应用场景
### interface：
1. 定义组件的props
2. 扩展接口（extend一个接口）
实际运用：
3. 表单数据
4. 组件状态数据

### type：
1. 联合类型
2. 基础类型的别名
3. **工具类型**，计算类型
4. 条件类型
5. 元祖类型
实际运用：
6. api请求参数
7. 状态枚举

**为啥API请求参数用type ，响应数据用interface？**
请求参数类型多变，比较灵活，也可用type实现类型联合
而响应数据结构是固定的


### 工具类型
```ts
// 1. Partial<T> - 将所有属性变为可选
interface User {
  id: string;
  name: string;
  email: string;
}
type PartialUser = Partial<User>;
// 等同于: { id?: string; name?: string; email?: string; }

// 2. Required<T> - 将所有属性变为必需
type RequiredUser = Required<PartialUser>;

// 3. Pick<T, K> - 选择指定属性
type UserBasic = Pick<User, 'id' | 'name'>;
// 等同于: { id: string; name: string; }

// 4. Omit<T, K> - 排除指定属性
type UserWithoutId = Omit<User, 'id'>;
// 等同于: { name: string; email: string; }

// 5. Record<K, T> - 创建键值对类型
type UserRoles = Record<string, boolean>;
// 等同于: { [key: string]: boolean; }

// 6. Exclude<T, U> - 从联合类型中排除
type Status = 'loading' | 'success' | 'error';
type NonLoadingStatus = Exclude<Status, 'loading'>;
// 等同于: 'success' | 'error'

// 7. Extract<T, U> - 从联合类型中提取
type SuccessStatus = Extract<Status, 'success' | 'complete'>;
// 等同于: 'success'

// ✅ 工具类型

type ApiResponse<T> = {

code: number;

data: T;

message: string;

};

const response = ref<ApiResponse<NoticeItem[]>>();
```
### 工具类型的使用(record< , >)
#### 1. 用户角色权限
```ts
// 定义用户角色权限
type UserRoles = Record<string, boolean>;

// 使用示例

const userPermissions: UserRoles = {

'read': true,

'write': false,

'delete': true,

'admin': false

};
// 检查权限
function hasPermission(permission: string): boolean {
return userPermissions[permission] || false;
}
console.log(hasPermission('read')); // true

console.log(hasPermission('write')); // false
```

#### 2. 加载状态管理
```ts
// 定义用户角色权限

type UserRoles = Record<string, boolean>;

  

// 使用示例

const userPermissions: UserRoles = {

'read': true,

'write': false,

'delete': true,

'admin': false

};

  

// 检查权限

function hasPermission(permission: string): boolean {

return userPermissions[permission] || false;

}

  

console.log(hasPermission('read')); // true

console.log(hasPermission('write')); // false
```

#### 3. 表单验证错误
```ts
// 表单字段错误信息

type FormErrors = Record<string, string>;

  

const errors: FormErrors = {

'username': '用户名不能为空',

'password': '密码长度至少6位',

'email': '邮箱格式不正确'

};

  

// 获取错误信息

function getError(field: string): string | undefined {

return errors[field];

}
```

#### 4. 配置对象
```ts
// API 端点配置

type ApiEndpoints = Record<string, string>;

  

const endpoints: ApiEndpoints = {

'users': '/api/users',

'books': '/api/books',

'notices': '/api/notices',

'activities': '/api/activities'

};

  

// 获取 API 地址

function getApiUrl(resource: string): string {

return endpoints[resource] || '/api/default';

}
```

#### 5. 枚举值映射
```ts
// 状态码对应的消息

type StatusMessages = Record<number, string>;

  

const statusMessages: StatusMessages = {

200: '请求成功',

400: '请求参数错误',

401: '未授权',

403: '禁止访问',

404: '资源不存在',

500: '服务器错误'

};

  

// 获取状态消息

function getStatusMessage(code: number): string {

return statusMessages[code] || '未知错误';

}
```
### 组件类型的使用：
```ts
// 1. 基础 Props 定义

interface ButtonProps {

type?: 'primary' | 'secondary' | 'danger';

size?: 'small' | 'medium' | 'large';

disabled?: boolean;

loading?: boolean;

onClick?: (event: MouseEvent) => void;

}

  

// 2. 在组件中使用

<script setup lang="ts">

// 方式一：使用 defineProps

const props = defineProps<ButtonProps>();

  

// 方式二：带默认值

const props = withDefaults(defineProps<ButtonProps>(), {

type: 'primary',

size: 'medium',

disabled: false,

loading: false

});

</script>
```

## 索引类型查询操作符keyof
用于获取对象类型的所有键名组成的联合类型。
```ts
interface User {
  id: number;
  name: string;
  email: string;
}

type UserKeys = keyof User; // "id" | "name" | "email"
```
1. 动态属性访问
```ts
function getValue<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { id: 1, name: "张三" };
const name = getValue(user, "name"); // 类型安全的属性访问
```

2. 创建映射类型
映射类型：
映射类型 是TypeScript中基于已有类型创建新类型的机制，通过遍历原类型的键来构造新的类型结构。
```ts
// 工具类型原理
type Partial<T> = { [K in keyof T]?: T[K] };
type OptionalUser = Partial<User>; // 所有属性变可选
- 1.keyof T - 获取类型T的所有键名
- 2.K in keyof T - 遍历T的每个键
- 3.?: - 为每个属性添加可选修饰符
- 4.T[K] - 保持原属性的值类型不变

// 自定义映射类型
type MappedType<T> = { [K in keyof T]: T[K] }

```

3. 约束泛型参数
```ts
function updateUser<K extends keyof User>(key: K, value: User[K]) {
  // key 只能是 User 的有效属性名
}
```