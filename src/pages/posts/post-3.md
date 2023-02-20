### 特性

#### 一、setup

- 包含一切数据，方法，计算属性，监听
- 不用加this
- 属性，方法 return
- setup不能是一个async函数，因为返回值不再是return的对象，而是promise，模板看不到return对象的属性。（也可以返回一个Promise实例，但需要Suspense和异步组件配合）
- 执行比beforeCreate()要早
- 参数
  - props：值为对象，包含：组件外部传递过来，但没有在props配置中声明的属性，相当于`this.$attrs`
  - slots：收到的插槽内容，相当于`this.$slots`
  - emit：分发自定义事件的函数，相当于`this.$emit`

#### 二、ref 函数

```vue
import { ref } from 'vue';
```



- 给**基本类型**属性添加响应式

  ```vue
  let name = ref('zy');
  let age = ref(3);
  ```

- 返回的是一个RefImpl对象，对象中的value属性是挂在RefImpl原型上的，value中由getter和setter

  ```vue
  name.value = '鹰酱'  // 可以修改值
  ```

- 模板中无需加value, vue会读取value

- 修改**对象**类型响应式（不用，应该用reactive）

  ```vue
  // 添加对象类型响应式
  let job = ref({
    type: 'Chess Player',
    level: 'Career II'
  })
  // 修改对象类型响应式
  job.value.type = '棋手'; 
  job.value.level = '职业二段';
  ```

- vue3对于对象都封装成Proxy对象（window身上的）

总结

- ref对基本类型数据处理仍采用defineProperty.
- ref处理对象**求助**了vue3的reactive函数（里面封装了Proxy）

#### 三、reactive函数

```vue
import { reactive } from 'vue';
```



- 处理**数组，对象**类型，不能处理**基本类型**
- 内部基于ES6的Proxy实现（劫持数据），通过代理对象操作源对象内部数据进行操作。

```vue
let job = reactive({
  type: 'Chess Player',
  level: 'Career II',
  // 可以处理深层次的数据
  a: {
    b: {
      c: 3,
    }
  }
})
let hobbies = reactive(['吃鱼肉肠', '下围棋', '植物大战僵尸']);

function changeInfo() {
	job.type = '棋手';
	job.level = '职业二段';
	job.a.b.c = 6;
	hobbies[0] = '喝饮料';
	hobbies[1] = '十四段扇子';
	hobbies[2] = '唱歌';
}
```

用reactive对以上基础类型+对象类型案例的对象写法：

```vue
// 定义一个person对象
let ZY = reactive({
  name: 'zy',
  age: 3,
  job: {
    type: 'Chess Player',
    level: 'Career II',
  },
  hobbies: ['吃鱼肉肠', '下围棋', '植物大战僵尸'],
})

function changeInfo() {
    ZY.name = '鹰酱';
    ZY.age = 18; 
    ZY.job.type = '棋手';
    ZY.job.level = '职业二段';
    ZY.hobbies[0] = '喝饮料';
    ZY.hobbies[1] = '十四段扇子';
    ZY.hobbies[2] = '唱歌';
}

return {
	ZY,
	changeInfo
}
```

#### 四、computed

- `import {computed} from 'vue';`

- 在setup里:

  ```vue
  let a = computed({
  	get() {
  		return ...;
  	},
  	set(value) {
  		// todo
  	}
  })
  ```

#### 五、watch

- 监视ref定义的一个响应式数据

  ```vue
  watch(sum, (newValue, oldValue) => {// todo}, {immediate: true});
  ```

- 监视ref定义的多个响应式数据

  ```vue
  watch([sum, msg], (newValue, oldValue) => {// todo}, {immediate: true	});
  ```

- 监视reactive定义的一个响应式数据的全部属性

  - oldValue在reactive中不便获取  (可以用ref带到外部监听) 

  - 强制开启了深度监视（deep配置无效）

    ```vue
    let person = reactive({
      name: '张三',
      age: 18,
      job: {
        type: {
          name: 'engineer'
        }
      }
    })
    
    // todo: <button @click="person.name += '~'">click me</button>
    // 情况3,注意和特殊情况作出区分
    watch(person, (newValue, oldValue) => {
      console.log('person变化了');
      console.log(newValue, oldValue);
    }, {deep: false}) // 此处deep配置无效
    ```

- 监视reactive定义的一个响应式数据中的某个属性

- 监视reactive定义的一个响应式数据中的多个属性

  ```vue
  let person = reactive({
    name: '张三',
    age: 18,
    job: {
      type: {
        name: 'engineer'
      }
    }
  })
  
  // 监听对象中的某个属性用函数的形式
  watch(()=>person.age, (newValue, oldValue) => {
    console.log('person中的age变化了');
    console.log(newValue, oldValue);
  })
  // 监听对象中的多个属性用函数的形式
  watch([()=>person.age, ()=>person.name], (newValue, oldValue) => {
    console.log('person中的age或name变化了');
    console.log(newValue, oldValue);
  })
  ```

- 特殊情况

  ```vue
  let person = reactive({
    name: '张三',
    age: 18,
    job: {
      type: {
        name: 'engineer'
      }
    }
  })
  
  // todo: <button @click="person.job.type.name += '~'">click job</button>
  
  watch(()=>person.job, (newValue, oldValue) => {
    console.log('person中的job变化了');
    console.log(newValue, oldValue);
  }, {deep: true}) // 和情况3作区分:此处需要开启深度监视
  
  // 总结：监视的是reactive中某个属性的值依然是个对象, 需要手动开启深度监视。
  ```

#### 六、watchEffect

```vue
watchEffect(() => {
  // watchEffect 比较智能，监视函数体中有的值。在这里监视sum和person.job.type.name
  const x1 = sum.value
  const x2 = person.job.type.name
  console.log('watchEffect所指定的回调执行了');
})
```

#### 七、hook

- 本质是一个函数，把setup函数中使用的Composition(属性、对象、计算属性、监听生命周期钩子) API进行了封装
- 复用代码，让setup中的逻辑更清楚易懂。
- 类似vue2.x中的mixin

#### 八、toRef & toRefs

- `import { toRef, toRefs } from 'vue'`

- 本质：创建一个ref对象，其value值指向另一个对象中的某个属性

  ```vue
  let girl = reactive({
     name: 'zy',
     age: 3,
     job: {
       type: 'Chess Player',
       level: 'Career II',
     }
  })
  
  return {
  	// 1. toRef
  	name: toRef(girl, 'name'),
  	age: toRef(girl, 'age'),
  	// 2. toRefs 指向对象第一层所有的属性,可以拿到name, age, job。如果是type,level，则需要job.type和job.level
  	// ...可以将所有的girl属性摊出来
  	...toRefs(girl)
  }
  <template> 
  	<h2> {{ name }} </h2>
  	<h2> {{ age }} </h2>
  </template>
  ```



#### 九、shallowReactive

- `import {shallowReactive} from 'vue'`

```vue
let p = shallowReactive({
    name: 'zy',
    age: 3,
    job: {
      type: 'Chess Player',
      level: 'Career II',
    }
})
<template> 
	<h2> {{ name }} </h2>
	<h2> {{ age }} </h2>
	<h2>职业：{{ job.type }}</h2> 
	<button @click="job.type += '~'">click me!!</button> // 操作无效
</template>
```

- 只处理对象最外层属性的响应式（浅响应式）。

#### 十、shallowRef

#### 十一、readonly与shallowReadonly

- `import { readonly, shallowReadonly } from 'vue'`

  ```vue
  girl = readonly(girl);
  ```

- readonly将响应式的数据设置为只读不能改，shallowReadonly只处理对象最外层属性。

- 应用场景：不希望数据被修改时

#### 十二、toRaw

- `import { toRaw } from 'vue';`

  ```vue
  let girl = reactive({})
  const p = toRaw(girl)
  ```

- 作用：将一个由`reactive`生成的**响应式对象**转为**普通对象**

- 应用场景：用于读取响应式对象对应的普通对象，对这个普通对象的所有操作，不会引起页面更新。

#### 十三、markRaw

- 作用：标记一个对象，使其**永远不会再成为响应式对象**

- 应用场景：

  - 有些值不应被设置为响应式的，例如**复杂的第三方类库**等。
  - 当渲染具有不可变数据源的大列表时，跳过响应式转换可以提高性能。

  ```vue
  import { markRaw } from 'vue';
  girl.fans = markRaw(fans);
  ```

#### 十四、customRef

- 自定义ref

  ```vue
  // 自定义一个ref-名为：myRef
  function myRef(value, delay) {
    let timer;
    console.log('--myRef--', value);
    return customRef((track, trigger) => {
      return {
        get() {
          console.log(`有人从myRef这个容器中读取数据了, 给出${value}`);
          track(); // 通知vue追踪value的变化
          return value;
        },
        set(newValue) {
          console.log(`有人把myRef这个容器中数据改为了:${newValue}`);
          // 防抖
          clearTimeout(timer);
          timer = setTimeout(() => {
            console.log('等1s');
            value = newValue;
            trigger(); // 通知vue重新解析模板
          }, delay);
        }
      }
    })
  }
  
  let keyWord = myRef('hello', 500); // 使用程序员自定义的ref
  ```

#### 十五、provide和inject

- provide

  - 应用场景：祖孙传递。在祖组件上使用，有跨层作用

    ```vue
    import { provide } from 'vue'
    setup() {
    	let car = reactive({
          name: '奔si',
          price: 200
        })
    
        provide('inherit', car);
    
        return {
          car
        }
    }
    ```

- inject

  - 接收provide传来的

    ```vue
    import { inject } from 'vue';
    export default {
      name: 'Son',
      setup() {
        const car = inject('inherit');
        return {
          car
        }
      }
    }
    ```

#### 十六、响应式数据的判断

- isRef：检查一个值是否为一个ref对象
- isReactive：检查一个对象是否由`reactive`创建的响应式代理
- isReadonly：检查一个对象是否是由`readonly`创建的只读代理
- isProxy：检查一个对象是否是由`reactive`或者`readonly`方法创建的代理

### Vue3.0中响应原理

#### vue2.x的响应式

- 实现原理

  - 对象类型：通过`Object.defineProperty()`对属性的读取、修改进行拦截（数据劫持）。

  - 数组类型：通过重写更新数组的一系列方法来实现拦截。（对数组的变更方法进行包裹）。

    ```javascript
    Object.defineProperty(data, 'count', {
        get(){},
        set(){}
    })
    ```

- 存在问题

  - 新增属性、删除属性，界面不会更新
  - 直接通过下标修改数组，界面不会更新

- 更新数据方式

  ```vue
  data() {
  	return {
  		person: {
  			name: '张三',
  			age: 18,
  			hobby: ['吃饭', '睡觉']
  		}
  	}
  }
  methods: {
  	addSex() {
  		// 一
  		this.$set(this.person, 'sex', '女');
  		// 二 
  		Vue.set(this, 'sex', '女');
  	},
  	deleteName() {
  		// 一
  		this.$delete(this.person, 'name');
  		// 二 
  		Vue.delete(this.person, 'name');
  	},
  	updateHobby() {
  		// 一
  		this.$set(this.person.hobby, 0, '逛街');
  		// 二 splice() 方法通过删除或替换现有元素或者原地添加新的元素来修改数组，并以数组形式返回被修改的内容。此方法会改变原数组。
  		this.person.hobby.splice(0, 1, '逛街');
  	}
  }
  ```

- Object.defineProperty模拟响应式实现

  ```vue
  let person = {
  	name: '张三',
  	age: 18
  }
  
  //模拟vue2中实现响应式
  let p = {} // vue2上来先准备一个空对象
  Object.defineProperty(p, 'name', {
      get() {
          console.log('有人读取了name属性, 做一些事情');
          return person.name;
      },
      set(value) {
          console.log('有人修改了name属性, 做一些事情');
          person.name = value;
      }
  });
  Object.defineProperty(p, 'age', {
      get() {
          console.log('有人读取了age属性, 做一些事情');
          return person.age;
      },
      set(value) {
          console.log('有人修改了age属性, 做一些事情');
          person.age = value;
      }
  }) 
  ```


#### vue3的响应式

- window.Proxy模拟实现（操作原数据）

  ```vue
  // 模拟vue3中实现响应式
  // Reflect也是可以获取，修改对象属性的一种方式
  // Reflect.get(obj, 'a'); 
  // Reflect.set(obj, 'a', 666);
  // Reflect.defineProperty()
  // 用Reflect的好处是容易调试，找到代码的问题。
  const p = new Proxy(person, {
      // get 有人读取p的某个属性时调用
      get(target, prop) {
          // 在target对象中找形参prop对应的属性
          // 用中括号的原因是：prop是形参，如果用点，则是去找名称为prop的属性
          console.log(`有人读取了p身上的${prop}属性`);
          // return target[prop];
  		return Reflect.get(target, prop);
      },
      // set 不仅修改属性时调用，增加属性时也会调用
      set(target, prop, value) {
          // target[prop] = value;
  		Reflect.set(target, prop, value);
          console.log(`有人修改了p身上的${prop}属性,修改后的值为${target[prop]}`);
      },
      // deleteProperty 有人删除p的某个属性时调用
      deleteProperty(target, prop) {
          console.log(`有人删除了p身上的${prop}属性`);
          // return delete target[prop];
  		return Reflect.deleteProperty(target, propName);
      }
  })
  ```

- 实现原理

  - 通过Proxy（代理）：拦截对象中任意属性的变化，包括：属性值的读写、属性的添加、属性的删除等。
  - 通过Reflect（反射）：对被代理对象的属性进行操作。

### 新的组件

#### Fragment

- Vue2中：组件必须有一个根标签
- Vue3中：组件可以没有根标签，内部会将多个标签包含在一个Fragment虚拟元素中
- 好处：减少标签层级，减小内存引用

#### teleport

- 应用场景：弹窗，实现传送的效果

  ```vue
  <teleport to='#dialog'>
  	<div v-if="isShow" class="mask">
  	  <div class="dialog">
  	    <h3>我是弹窗</h3>
  	    <h4>something</h4>
  	    <h4>something</h4>
  	    <h4>something</h4>
  	    <h4>something</h4>
  	    <button @click="isShow = false">关闭弹窗</button>
  	  </div>
  	</div>
  </teleport>
  <!-index.html中->
  <div id="app"></div>
  <div id="dialog"></div>
  ```

#### Suspense

- 组件静态引入

  ```vue
  import Child from './components/Child.vue';
  ```

  - 概念：所有组件加载完，一块显示

- 组件动态(异步)引入

  ```vue
  import { defineAsyncComponent } from 'vue';
  const Child = defineAsyncComponent(() => import('./components/Child.vue'))
  ```

  - 概念：加载完一个组件显示一个组件

- Suspense的应用

  ```vue
  <Suspense>
      <template v-slot:default>
  	  // 正常加载的组件
        <Child/>
      </template>
      <template v-slot:fallback>
  	  // 由于某种原因，组件加载慢了的处理方法
        <h3>稍等.加载中...</h3>
      </template>
  </Suspense>
  ```

- 让组件等一等的方法

  ```vue
  async setup() {
  	let p = new Promise((resolve, reject) => {
  		setTimeout(() => {
  			resolve({sum});
  		}, 3000)
  	})
  	return await p
  }
  ```