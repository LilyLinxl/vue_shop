注意点和错误
1.  <el-breadcrumb-item :to="{ path:'/home'}">首页</el-breadcrumb-item>
:to="{ path:'/home'}"

2.调用后台接口获取不到数据
```javascript
async getGoodsList(){
      const {data:res} = await this.$http.get('goods',this.queryList)
      console.log(res.data)
    }
```
传递参数的方式出错了,应该是
```javascript
 const {data:res} = await this.$http.get('goods',{
      params: this.queryList})
```
3.搜索框和搜索，添加按钮的布局
Row 组件 提供 gutter 属性来指定每一栏之间的间隔，默认间隔为 0。
通过 row 和 col 组件，并通过 col 组件的 span 属性我们就可以自由地组合布局。
span：栅格布局
gutter：分栏间隔

3.输入框中嵌入按钮的方法
可通过 slot 来指定在 input 中前置或者后置内容。
```html
<el-input placeholder="请输入内容" v-model="input3" class="input-with-select">
    <el-button slot="append" icon="el-icon-search"></el-button>
</el-input>
  ```
4.点击按钮跳转到其它组件怎么做
按钮中添加点击事件，事件调用跳转方法，方法中写跳转的代码
```javascript
 goAddpage() {
      this.$router.push('/goods/add')
    }
```
