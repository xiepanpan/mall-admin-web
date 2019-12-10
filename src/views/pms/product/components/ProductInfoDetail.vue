<template>
  <div style="margin-top: 50px">
    <el-form :model="value" :rules="rules" ref="productInfoForm" label-width="120px" style="width: 600px" size="small">
      <el-form-item label="商品分类：" prop="productCategoryId">
        <el-cascader
          ref="productCateCascaderEle"
          v-model="selectProductCateValue"
          :options="productCateOptions"
          :props="optionProps">
        </el-cascader>
      </el-form-item>
      <el-form-item label="商品名称：" prop="name">
        <el-input v-model="value.name"></el-input>
      </el-form-item>
      <el-form-item label="副标题：" prop="subTitle">
        <el-input v-model="value.subTitle"></el-input>
      </el-form-item>
      <el-form-item label="商品品牌：" prop="brandId">
        <el-select
          v-model="value.brandId"
          @change="handleBrandChange"
          placeholder="请选择品牌">
          <el-option
            v-for="item in brandOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="商品介绍：">
        <el-input
          :autoSize="true"
          v-model="value.description"
          type="textarea"
          placeholder="请输入内容"></el-input>
      </el-form-item>
      <el-form-item label="商品货号：">
        <el-input v-model="value.productSn"></el-input>
      </el-form-item>
      <el-form-item label="商品售价：">
        <el-input v-model="value.price"></el-input>
      </el-form-item>
      <el-form-item label="市场价：">
        <el-input v-model="value.originalPrice"></el-input>
      </el-form-item>
      <el-form-item label="商品库存：">
        <el-input v-model="value.stock"></el-input>
      </el-form-item>
      <el-form-item label="计量单位：">
        <el-input v-model="value.unit"></el-input>
      </el-form-item>
      <el-form-item label="商品重量：">
        <el-input v-model="value.weight" style="width: 300px"></el-input>
        <span style="margin-left: 20px">克</span>
      </el-form-item>
      <el-form-item label="排序">
        <el-input v-model="value.sort"></el-input>
      </el-form-item>
      <el-form-item style="text-align: center">
        <el-button type="primary" size="medium" @click="handleNext('productInfoForm')">下一步，填写商品促销</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
  import {fetchListWithChildren} from '@/api/productCate'
  import {fetchList as fetchBrandList} from '@/api/brand'
  import {getProduct} from '@/api/product';

  export default {
    name: "ProductInfoDetail",
    props: {
      value: Object,
      isEdit: {
        type: Boolean,
        default: false
      }
    },
    data() {
      return {
        optionProps:{
          value: 'id',
          label: 'name',
          children: 'children'
        },
        hasEditCreated:false,
        //选中商品分类的值
        selectProductCateValue: [],
        productCateOptions: [],
        brandOptions: [],
        rules: {
          name: [
            {required: true, message: '请输入商品名称', trigger: 'blur'},
            {min: 2, max: 140, message: '长度在 2 到 140 个字符', trigger: 'blur'}
          ],
          subTitle: [{required: true, message: '请输入商品副标题', trigger: 'blur'}],
          productCategoryId: [{required: true, message: '请选择商品分类', trigger: 'blur'}],
          brandId: [{required: true, message: '请选择商品品牌', trigger: 'blur'}],
          description: [{required: true, message: '请输入商品介绍', trigger: 'blur'}],
          requiredProp: [{required: true, message: '该项为必填项', trigger: 'blur'}]
        }
      };
    },
    async created() {
      this.getBrandList();
      this.productCateOptions = await this.getProductCateList();
      // console.log(this.productCateOptions, '1');

    },
    computed:{
      //商品的编号
      productId(){
        return this.value.id;
      }
    },
    watch: {
      productId:async function(newValue){
        if(!this.isEdit)return;
        if(this.hasEditCreated)return;
        if(newValue===undefined||newValue==null||newValue===0)return;

        console.log("watch...productId...",newValue, '2');
        this.productCateOptions = await this.getProductCateList();
        this.handleEditCreated();
      },
      selectProductCateValue: function (newValue) {
        let a = new Array(); a.push(this.$refs['productCateCascaderEle'].currentLabels)
        this.value.productCategoryId = newValue[newValue.length-1];
        this.value.productCategoryName= a.join(",");
        //this.getCateNameById(this.value.productCategoryId);
         console.log("绑定选中的三级分类...",newValue);
      }
    },
    methods: {
      //处理后台兄弟传来children:[]（空数组），显示空白的bug
      getTreeData(data){
                // 循环遍历json数据
                for(var i=0;i<data.length;i++){
                    
                    if(data[i].children.length<1){
                        // children若为空数组，则将children设为undefined
                        data[i].children=undefined;
                    }else {
                        // children若不为空数组，则继续 递归调用 本方法
                        this.getTreeData(data[i].children);
                    }
                }
                return data;
        },
      //处理编辑逻辑
      handleEditCreated(){
        console.log("handleEditCreated....",this.productCateOptions);
        if(this.value.productCategoryId!=null){
          for(let i=0;i<this.productCateOptions.length;i++){
            //遍历所有的分类。找到当前产品所属的完整分类进行回显
            if(this.productCateOptions[i].children){
              for(let j=0;j<this.productCateOptions[i].children.length;j++){
                //找到了二级分类的id
                if(this.productCateOptions[i].children[j].id == this.value.cateParentId){
                  console.log("找到了三级分类的完整进行进行回显")
                  this.selectProductCateValue.push(this.productCateOptions[i].id);
                  this.selectProductCateValue.push(this.value.cateParentId);
                  this.selectProductCateValue.push(this.value.productCategoryId);
                }
              }
            }
           
          }

          //以上运行了如果数组还是空，说明是以前的二级分类
          if(this.selectProductCateValue.length==0){
            console.log("只有二级分类的兼容回显");
            this.selectProductCateValue.push(this.value.cateParentId);
            this.selectProductCateValue.push(this.value.productCategoryId);
          }  
         
        }
        this.hasEditCreated=true;
      },
      getProductCateList() {
        return new Promise((res, reject) => {
          console.log("getProductCateList...：",this.productCateOptions);
            
              fetchListWithChildren().then(response => {
                  let list = response.data;
                 //  this.productCateOptions = this.getTreeData(list);
                  res(this.getTreeData(list))
                  console.log('3')
                  // for (let i = 0; i < list.length; i++) {
                  //   let children = [];
                  //   if (list[i].children != null && list[i].children.length > 0) {
                  //     for (let j = 0; j < list[i].children.length; j++) {
                  //       let chil = list[i].children[j].children;
                  //       console.log("最内三级分类...",chil);

                  //       children.push({label: list[i].children[j].name, value: list[i].children[j].id,children:chil});
                  //     }
                  //   }
                  //   this.productCateOptions.push({label: list[i].name, value: list[i].id, children: children});
                  // }
                  console.log("整理后的三级分类：",this.productCateOptions);
            });
        
        })
      },
      getBrandList() {
        fetchBrandList({pageNum: 1, pageSize: 100}).then(response => {
          this.brandOptions = [];
          let brandList = response.data.list;
          for (let i = 0; i < brandList.length; i++) {
            this.brandOptions.push({label: brandList[i].name, value: brandList[i].id});
          }
        });
      },
      // getCateNameById(id){
      //   console.log("能不能获取到商品信息?",this.value)

      //   return "外套";
      // },
      handleNext(formName){
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.$emit('nextStep');
          } else {
            this.$message({
              message: '验证失败',
              type: 'error',
              duration:1000
            });
            return false;
          }
        });
      },
      handleBrandChange(val) {
        let brandName = '';
        for (let i = 0; i < this.brandOptions.length; i++) {
          if (this.brandOptions[i].value === val) {
            brandName = this.brandOptions[i].label;
            break;
          }
        }
        this.value.brandName = brandName;
      }
    }
  }
</script>

<style scoped>
</style>
