<template>
    <div>
      <v-card>
        <!-- 卡片的头部 -->
        <v-card-title>
          <v-btn color="success" @click="addBrand">新增品牌</v-btn>
          <!--空间隔离组件-->
          <v-spacer />
          <!--搜索框，与search属性关联-->
          <v-text-field label="输入关键字搜索" v-model="search" append-icon="search" hide-details/>
        </v-card-title>
        <!-- 分割线 -->
        <v-divider/>
        <!--卡片的中部-->
        <v-data-table
          :headers="headers"
          :items="brands"
          :search="search"
          :pagination.sync="pagination"
          :total-items="totalBrands"
          :loading="loading"
          class="elevation-1"
        >
          <template slot="items" slot-scope="props">
            <td>{{ props.item.id }}</td>
            <td class="text-xs-center">{{ props.item.name }}</td>
            <td class="text-xs-center"><img :src="props.item.image"></td>
            <td class="text-xs-center">{{ props.item.letter }}</td>
            <td class="justify-center layout">
              <!--通过props.item将父组件的值传递给子组件-->
              <v-btn color="info" @click="editBrand(props.item)">编辑</v-btn>
              <v-btn color="warning" @click="removeBrand(props.item.name)">删除</v-btn>
            </td>
          </template>
        </v-data-table>
      </v-card>
      <!--弹出的对话框-->
      <v-dialog max-width="500" v-model="show" persistent>
        <v-card>
          <!--对话框的标题-->
          <v-toolbar dense dark color="primary">
            <v-toolbar-title>{{ isEdit ? '修改':'新增'}}品牌</v-toolbar-title>
            <v-spacer/>
            <!--关闭窗口的按钮-->
            <v-btn icon @click="closeWindow"><v-icon>close</v-icon></v-btn>
          </v-toolbar>
          <!--对话框的内容，表单-->
          <v-card-text class="px-5">
            <!--引入自己的组件，并且将父组件的值传递给子组件-->
            <my-brand-form @close="closeWindow" :oldBrand="oldBrand" :isEdit="isEdit"/>
          </v-card-text>
        </v-card>
      </v-dialog>
      <!--弹出的删除对话框-->
      <v-dialog max-width="500" v-model="deleteShow" persistent>
        <v-card>
          <!--对话框的标题-->
          <v-toolbar dense dark color="error">
            <v-toolbar-title color="error">删除品牌</v-toolbar-title>
            <v-spacer/>
            <!--关闭窗口的按钮-->
            <v-btn icon @click="closeWindow"><v-icon>close</v-icon></v-btn>
          </v-toolbar>
          <!--对话框的内容，表单-->
          <v-card-text class="px-5">
            <v-text-field v-model="brandName" label="是否要删除品牌" required />
            <v-layout class="my-4" row>
              <v-spacer/>
              <v-btn @click="submit" color="primary">确定</v-btn>
              <v-btn @click="clear" >取消</v-btn>
            </v-layout>
          </v-card-text>
        </v-card>
      </v-dialog>
    </div>
</template>

<script>
  // 导入自定义的表单组件
  import MyBrandForm from './MyBrandForm'

  export default {
    name: "my-brand",
    data() {
      return {
        search: '', // 搜索过滤字段
        totalBrands: 0, // 总条数
        brands: [], // 当前页品牌数据
        loading: true, // 是否在加载中
        pagination: {}, // 分页信息
        headers: [
          {text: 'id', align: 'center', value: 'id'},
          {text: '名称', align: 'center', sortable: false, value: 'name'},
          {text: 'LOGO', align: 'center', sortable: false, value: 'image'},
          {text: '首字母', align: 'center', value: 'letter', sortable: true,},
          {text: '操作', align: 'center', value: 'id', sortable: false}
        ],
        show: false,  //控制弹出对话框的显示
        deleteShow: false, //控制删除弹出框
        oldBrand:{},  //即将被编辑的品牌数据
        isEdit:false, //是否是编辑
        brandName:'', //商品名称
      }
    },
    //钩子函数
    mounted(){ // 渲染后执行
      // 查询数据
      this.getDataFromServer();
    },
    watch:{ //监视属性的变化pagination
      pagination: {
        deep:true, //deep为true会监视pagination的属性及属性中对象属性的变化
        handler(){
          //变化后的回调函数，这里我们再次调用getDataFromServer函数重新加载数据即可
          this.getDataFromServer();
        }
      },
      search:{ //监控搜索字段
        handler(){
          this.getDataFromServer();
        }
      }

    },
    methods:{
      submit(){
        this.deleteShow = false;
        // 重新加载数据
        this.getDataFromServer();
      },
      removeBrand(name){
        this.deleteShow = true;  //删除弹出框显示
        this.brandName = name;
      },
      getDataFromServer(){ // 从服务的加载数的方法。

        this.$http.get("/item/brand/page",{
          params:{
            key: this.search, // 搜索条件
            page: this.pagination.page,// 当前页
            rows: this.pagination.rowsPerPage,// 每页大小
            sortBy: this.pagination.sortBy,// 排序字段
            desc: this.pagination.descending// 是否降序
          }
        }).then(resp => { // 请求成功的回调，这里使用箭头函数
          // 将得到的数据赋值给本地属性
          this.brands = resp.data.items;
          this.totalBrands = resp.data.total;
          // 完成赋值后，把加载状态赋值为false
          this.loading = false;
        }).catch(reason => {  //请求失败的回调

        })
      },
      addBrand(){
        this.isEdit = false;
        // 控制弹窗可见：
        this.show = true;
        // 把oldBrand变为null
        this.oldBrand = null;
      },
      editBrand(oldBrand){
        this.$http.get("/item/category/bid/" + oldBrand.id)
          .then(({data})=>{
            this.isEdit = true;
            // 控制弹窗可见：
            this.show = true;
            // 获取要编辑的brand
            this.oldBrand = oldBrand;
            // 回显商品分类
            this.oldBrand.categories = data;
          })
      },
      closeWindow(){
        // 关闭窗口
        this.show = false;
        // 重新加载数据
        this.getDataFromServer();
      },
    },
    components:{    //添加一个组件进来
      MyBrandForm
    }
  }
</script>

<style scoped>

</style>
