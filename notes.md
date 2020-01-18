需要注意的步骤
1.提交表单数据前需要验证表单
2.关闭对话框后清除表单域的方法是
 this.$refs.editCateFormRef.resetFields()

 格式化用prettier，如果修改配置没有反应，重启编辑器试试看

3.nextTick的用法
在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。

4.在ES6 / Typescript中将_（下划线）变量与箭头函数一起使用
_ => console.log('Not using any parameters');

我了解变量_表示不在乎/不使用，但是由于它是唯一的变量，因此没有任何理由更喜欢使用_：

() => console.log('Not using any parameters');

当然，键入的字符数不能少于一个。在我看来，（）语法可以更好地传达意图，并且还更具体地说明类型，因为否则我认为第一个示例应该看起来像这样：

(_: any) => console.log('Not using any parameters');

万一重要，这是使用它的上下文：

submit(query: string): void {

    this.router.navigate(['search'], { queryParams: { query: query } })

      .then(_ => this.search());

}

5.$refs
访问子组件实例或子元素
尽管存在 prop 和事件，有的时候你仍可能需要在 JavaScript 里直接访问一个子组件。为了达到这个目的，你可以通过 ref 这个 attribute 为子组件赋予一个 ID 引用。例如：

<base-input ref="usernameInput"></base-input>
现在在你已经定义了这个 ref 的组件里，你可以使用：

this.$refs.usernameInput
来访问这个 <base-input> 实例，以便不时之需。比如程序化地从一个父级组件聚焦这个输入框。在刚才那个例子中，该 <base-input> 组件也可以使用一个类似的 ref 提供对内部这个指定元素的访问，例如：

<input ref="input">
甚至可以通过其父级组件定义方法：

methods: {
  // 用来从父级组件聚焦输入框
  focus: function () {
    this.$refs.input.focus()
  }
}
这样就允许父级组件通过下面的代码聚焦 <base-input> 里的输入框：

this.$refs.usernameInput.focus()
当 ref 和 v-for 一起使用的时候，你得到的引用将会是一个包含了对应数据源的这些子组件的数组。

**$refs 只会在组件渲染完成之后生效，并且它们不是响应式的。这仅作为一个用于直接操作子组件的“逃生舱”——你应该避免在模板或计算属性中访问 $refs。**

6.Duplicate keys detected: 'tab-2'. This may cause an update error.
el-tabs中有name相同的el-tab-pane

7."TypeError: Cannot read property '$options' of undefined"
解决方案同下
8.Injection "elForm" not found
未找到注入“elForm”
我的错误原因是：直接写了el-form-item而没用 el-form包裹住
found in
9.Property or method "elForm" is not defined on the instance but referenced during render. Make sure that this property is reactive, either in the data option, or for class-based components, by initializing the property. 
解决方案同上


10.预览图片时，图片的尺寸能不能适应预览框的大小
图片设置样式：width:100%
