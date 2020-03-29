@[toc](vue+ElementUI 表单嵌套表格逐行校验（新增、编辑）的完美解决方案)

# 一、成果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200329212923318.gif#pic_center)

如图，ElementUI表单里嵌套了表格，表格内每行能进行新增、编辑、删除等操作，`同时能针对该行的字段进行校验（而不是整个表单校验！）`，这种需求应该很常见，但是搜了很多资料，没有完美的解决方案，大部分都只是针对整个表单进行校验，而不是一行里面的字段校验，所以困扰了很久，最近终于研究了完美解决方案，分享给大家。

<br/>

***
<br/>

# 二、要点和解决思路

1. 表格数据datas必须是某对象（form）的一个属性 
	```
	  form: {
	    datas: [
	      { id: 1, name: "张三", age:20 },
	      { id: 2, name: "李四", age:32 },
	    ],
	  },
	```

 2. **每个字段要动态绑定表单的prop属性（以name字段为例）**
	 ```
	 <el-form :model="form" :rules="rules" ref="form">
	 	.....
		<el-form-item :prop="'datas.'+scope.$index + '.name'" :rules='rules.name'>
	</el-form>
	```
	关键代码`:prop="'datas.'+scope.$index + '.name'"`，这是elementui规定的格式，渲染后的结果为'datas.1.name'。所以必须结合第1点才能实现，否则提示出错！（这里坑了我很长时间）

3. **每个字段要动态绑定表单的rules属性**
	<el-form-item :prop="'datas.'+scope.$index + '.name'" `:rules='rules.name'`>

4. **针对某一行的所有字段进行校验**
	通过控制台查看得知：`表单的字段对象存在this.$refs[‘form’].fields里面，并且字段对象具有prop属性（’datas.1.name’）和validateField方法（验证’datas.1.name’能否通过校验）。`

	那么我们创建一个函数validateField，来判断第index行的所有字段能否通过校验：
	```js
	  //对部分表单字段进行校验
	  validateField(form,index){
	    let result = true;
	    for (let item of this.$refs[form].fields) {
	      if(item.prop.split(".")[1] == index){
	        this.$refs[form].validateField(item.prop,(error)=>{
	          if(error!=""){
	            result = false;
	          }
	        });
	      }
	      if(!result) break;
	    }
	    return result;
	  },
	```
5. **针对某一行的所有字段进行重置操作**
	同理，表单的字段对象存在resetField方法来重置数据
	```js
	  //对部分表单字段进行重置
	  resetField(form,index){
	    this.$refs[form].fields.forEach(item=>{
	      if(item.prop.split(".")[1] == index){
	        item.resetField();
	      }
	    })
	  },
	```

6. **每一行的状态可以通过添加属性action来处理**
	数据初始化的时候，每一行对象添加action属性（'view'：查看状态，'edit'编辑状态，'add'新增状态）
	```js
	  created() {
	     //处理数据，为已有数据添加action:'view'
	     this.form.datas.map(item => {
	       this.$set(item,"action","view")
	       return item;
	     });
	
	     //再插入一条添加操作的数据
	     this.form.datas.unshift({
	       id:undefined,
	       name:undefined,
	       age:undefined,
	       action: "add"
	     });
	   },
	```

<br/>

***
<br/>

# 三、源码

查看“源码.vue”