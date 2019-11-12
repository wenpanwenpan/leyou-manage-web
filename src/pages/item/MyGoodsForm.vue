<template>
  <v-card>
    <v-stepper v-model="step">
      <v-stepper-header>
        <v-stepper-step :complete="step > 1" step="1">基本信息</v-stepper-step>
        <v-divider/>
        <v-stepper-step :complete="step > 2" step="2">商品描述</v-stepper-step>
        <v-divider/>
        <v-stepper-step :complete="step > 3" step="3">规格参数</v-stepper-step>
        <v-divider/>
        <v-stepper-step step="4">SKU属性</v-stepper-step>
      </v-stepper-header>
      <v-stepper-items>
        <v-stepper-content step="1">
          基本信息
          <v-cascader
            url="/item/category/list"
            required
            showAllLevels
            v-model="goods.categories"
            label="请选择商品分类"/>
          <!--品牌-->
          <v-select
            :items="brandOptions"
            item-text="name"
            item-value="id"
            label="所属品牌"
            v-model="goods.brandId"
            required
            autocomplete
            clearable
            dense chips
          />
          <v-text-field label="商品标题" v-model="goods.title" :counter="200" required />
          <v-text-field label="商品卖点" v-model="goods.subTitle" :counter="200"/>
          <v-text-field label="包装清单" v-model="goods.spuDetail.packingList" :counter="1000" multi-line :rows="3"/>
          <v-text-field label="售后服务" v-model="goods.spuDetail.afterService" :counter="1000" multi-line :rows="3"/>
        </v-stepper-content>
        <v-stepper-content step="2">
          商品描述
          <v-editor v-model="goods.spuDetail.description" upload-url="/upload/image"/>
        </v-stepper-content>
        <!--3、规格参数-->
        <v-stepper-content step="3">
          <v-flex class="xs10 mx-auto px-3">
            <!--遍历整个规格参数，获取每一组-->
            <v-card v-for="spec in specifications" :key="spec.group" class="my-2">
              <!--组名称-->
              <v-card-title class="subheading">{{spec.group}}</v-card-title>
              <!--遍历组中的每个属性，并判断是否是全局属性，不是则不显示-->
              <v-card-text v-for="param in spec.params" :key="param.k" v-if="param.global" class="px-5">
                <!--判断是否有可选项，如果没有，则显示文本框。还要判断是否是数值类型，如果是把unit显示到后缀-->
                <v-text-field v-if="param.options.length <= 0"
                              :label="param.k" v-model="param.v" :suffix="param.unit || ''"/>
                <!--否则，显示下拉选项-->
                <v-select v-else :label="param.k" v-model="param.v" :items="param.options"/>
              </v-card-text>
            </v-card>
          </v-flex>
        </v-stepper-content>
        <v-stepper-content step="4">
          SKU属性
        </v-stepper-content>
      </v-stepper-items>
    </v-stepper>
  </v-card>

</template>

<script>
  export default {
    name: "my-goods-form",
    props: {
      oldGoods: {
        type: Object
      },
      isEdit: {
        type: Boolean,
        default: false
      },
      step:{
        type: Number,
        default: 1
      },
    },
    data() {
      return {
        goods:{
          categories:{}, // 商品3级分类数组信息
          brandId: 0,// 品牌id信息
          title: '',// 标题
          subTitle: '',// 子标题
          spuDetail: {
            packingList: '',// 包装列表
            afterService: '',// 售后服务
          },
        },
        brandOptions: [],  //品牌列表
        specifications: [], //规格参数的模板
      }
    },
    methods: {
    },
    watch: {
      'goods.categories': {
        deep: true,
        handler(val) {
          // 判断商品分类是否存在，存在才查询
          if (val && val.length > 0) {
            // 根据分类查询品牌
            this.$http.get("/item/brand/cid/" + this.goods.categories[2].id)
              .then(({data}) => {
                this.brandOptions = data;
              })
            //根据分类查询规格参数
            this.$http.get("/item/spec/params?cid=" + this.goods.categories[2].id)
              .then(({data}) => {
                //保存全部规格
                this.specifications = data;
              })
          }
        }
      }
    }
  }
</script>

<style scoped>

</style>
