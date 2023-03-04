# python/django/vue问题记录

1. python继承的三种用法
2. uuid.uuid1, uuid4用法
3. django model中 DatetimeField 的auto_now, auto_now_add 用法区别
4. django model中 verbose_name，verbose_name_plural用法区别
5. django MEDIA_ROOT 和 MEDIA_URL配置用法
6. 正则表达式中的?P<name>命名组用法
7. receiver装饰器用法
8. django路由分发配置
9. flex布局
10. django中直接引入 vue.js
11. django中使用 vue.js 语法，直接在 html 中无法使用{{variable}} 方式填充数据（替代方案：delimiters: ["[[","]]"])
12. vue 冒号":"的作用 - 冒号即 v-bind，动态绑定；如\<a :href="..."> 等价于 \<a v-bind:href="..."> 
13. vue 显示隐藏 v-show 和 v-if 的用法
14. css 中&符号的作用 - sass 语法，标识嵌套的上一层
15. css 中>符号的作用 - 子元素选择器，用法：> div{...}
16. django 自定义过滤器
17. django 模板标签（simple_tag; inclusion_tag）