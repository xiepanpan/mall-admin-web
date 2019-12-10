<template>
  <el-card class="form-container" shadow="never">
    <el-form :model="productCate"
             :rules="rules"
             ref="productCateFrom"
             label-width="150px">
      <el-form-item label="分类名称：" prop="name">
        <el-input v-model="productCate.name"></el-input>
      </el-form-item>
      <el-form-item label="上级分类：">
        <el-cascader v-model="selectProductCateIds"
          :options="selectProductCateList"
          :props="optionProps"
          filterable
          change-on-select
        ></el-cascader>
      </el-form-item>
      <el-form-item label="数量单位：">
        <el-input v-model="productCate.productUnit"></el-input>
      </el-form-item>
      <el-form-item label="排序：">
        <el-input v-model="productCate.sort"></el-input>
      </el-form-item>
      <el-form-item label="是否显示：">
        <el-radio-group v-model="productCate.showStatus">
          <el-radio :label="1">是</el-radio>
          <el-radio :label="0">否</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="是否显示在导航栏：">
        <el-radio-group v-model="productCate.navStatus">
          <el-radio :label="1">是</el-radio>
          <el-radio :label="0">否</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="分类图标：">
        <single-upload v-model="productCate.icon"></single-upload>
      </el-form-item>
      <el-form-item v-for="(filterProductAttr, index) in filterProductAttrList"
                    :label="index | filterLabelFilter"
                    :key="filterProductAttr.key"
      >
        <el-cascader
          clearable
          v-model="filterProductAttr.value"
          :options="filterAttrs">
        </el-cascader>
        <el-button style="margin-left: 20px" @click.prevent="removeFilterAttr(filterProductAttr)">删除</el-button>
      </el-form-item>
      <el-form-item>
        <el-button size="small" type="primary" @click="handleAddFilterAttr()">新增</el-button>
      </el-form-item>
      <el-form-item label="关键词：">
        <el-input v-model="productCate.keywords"></el-input>
      </el-form-item>
      <el-form-item label="分类描述：">
        <el-input type="textarea" :autosize="true" v-model="productCate.description"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSubmit('productCateFrom')">提交</el-button>
        <el-button v-if="!isEdit" @click="resetForm('productCateFrom')">重置</el-button>
      </el-form-item>
    </el-form>
  </el-card>
</template>

<script>
  import {fetchListWithChildren} from '@/api/productCate'
  import {fetchList, createProductCate, updateProductCate, getProductCate} from '@/api/productCate';
  import {fetchListWithAttr} from '@/api/productAttrCate';
  import {getProductAttrInfo} from '@/api/productAttr';
  import SingleUpload from '@/components/Upload/singleUpload';

  // const 
  const defaultProductCate = {
    description: '',
    icon: '',
    keywords: '',
    name: '',
    navStatus: 0,
    parentId: 0,
    productUnit: '',
    showStatus: 0,
    sort: 0,
    productAttributeIdList: []
  };
  export default {
    name: "ProductCateDetail",
    components: {SingleUpload},
    props: {
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
        productCate: Object.assign({}, defaultProductCate),
        selectProductCateIds: [],
        selectProductCateList: [],
        rules: {
          name: [
            {required: true, message: '请输入品牌名称', trigger: 'blur'},
            {min: 2, max: 140, message: '长度在 2 到 140 个字符', trigger: 'blur'}
          ]
        },
        filterAttrs: [],
        filterProductAttrList: [{
          value: []
        }]
      }
    },
    created() {
      if (this.isEdit) {
        console.log("修改进行....")
        getProductCate(this.$route.query.id).then(response => {
          
          this.productCate = response.data;
          console.log("getProductCate....",this.productCate)

          

        });
        getProductAttrInfo(this.$route.query.id).then(response => {
          if (response.data != null && response.data.length > 0) {
            this.filterProductAttrList = [];
            for (let i = 0; i < response.data.length; i++) {
              this.filterProductAttrList.push({
                key: Date.now() + i,
                value: [response.data[i].attributeCategoryId, response.data[i].attributeId]
              })
            }
          }
        });
      } else {
        this.productCate = Object.assign({}, defaultProductCate);
      }
      this.getSelectProductCateList();
      this.getProductAttrCateList();
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
      getSelectProductCateList() {
         let editFlag = this.isEdit;
         fetchListWithChildren().then(response => {
                   this.selectProductCateList = this.getTreeData(response.data);
                   this.selectProductCateList.unshift({id: 0, name: '无上级分类'});
                   console.log("getSelectProductCateList....",this.selectProductCateList);


                   //分类全部查找完了，如果是修改，是需要回显的
                   if(editFlag){
                      let showIds = new Array();
                      let _id = this.productCate.id; //当前分类id；
                      let pid = this.productCate.parentId;//pid
                      console.log("他的父id是...",pid);

                      showIds.push(pid);//当前类的父id，先放入，无论是0还是n
                      //再在二级菜单，一级菜单中寻找这个父id对应的菜单是否还有父id
                    
                      for(let i=1;i<this.selectProductCateList.length;i++){
                        //跳过0号元素
                        for(let j=0;j<this.selectProductCateList[i].children.length;j++){
                          if(pid == this.selectProductCateList[i].children[j].id){
                            showIds.unshift(this.selectProductCateList[i].children[j].parentId);
                          }
                        }
                        if(pid == this.selectProductCateList[i].id){
                            if(this.selectProductCateList[i].parentId!=0)
                            showIds.unshift(this.selectProductCateList[i].parentId);
                          }
                      }
                      this.selectProductCateIds = showIds;
                      console.log("最终放的东西是...",showIds)
                   }
            });
      },
      getProductAttrCateList() {
        fetchListWithAttr().then(response => {
          let list = response.data;
          for (let i = 0; i < list.length; i++) {
            let productAttrCate = list[i];
            let children = [];
            if (productAttrCate.productAttributeList != null && productAttrCate.productAttributeList.length > 0) {
              for (let j = 0; j < productAttrCate.productAttributeList.length; j++) {
                children.push({
                  label: productAttrCate.productAttributeList[j].name,
                  value: productAttrCate.productAttributeList[j].id
                })
              }
            }
            this.filterAttrs.push({label: productAttrCate.name, value: productAttrCate.id, children: children});
          }
        });
      },
      getProductAttributeIdList() {
        //获取选中的筛选商品属性
        let productAttributeIdList = [];
        for (let i = 0; i < this.filterProductAttrList.length; i++) {
          let item = this.filterProductAttrList[i];
          if (item.value !== null && item.value.length === 2) {
            productAttributeIdList.push(item.value[1]);
          }
        }
        return productAttributeIdList;
      },
      onSubmit(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.$confirm('是否提交数据', '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning'
            }).then(() => {
              if (this.isEdit) {
                this.productCate.productAttributeIdList = this.getProductAttributeIdList();
                this.productCate.parentId = this.selectProductCateIds[this.selectProductCateIds.length-1];
                updateProductCate(this.$route.query.id, this.productCate).then(response => {
                  this.$message({
                    message: '修改成功',
                    type: 'success',
                    duration: 1000
                  });
                  this.$router.back();
                });
              } else {
                this.productCate.productAttributeIdList = this.getProductAttributeIdList();
                this.productCate.parentId = this.selectProductCateIds[this.selectProductCateIds.length-1];
                createProductCate(this.productCate).then(response => {
                  this.$refs[formName].resetFields();
                  this.resetForm(formName);
                  this.$message({
                    message: '提交成功',
                    type: 'success',
                    duration: 1000
                  });
                });
              }
            });

          } else {
            this.$message({
              message: '验证失败',
              type: 'error',
              duration: 1000
            });
            return false;
          }
        });
      },
      resetForm(formName) {
        this.$refs[formName].resetFields();
        this.productCate = Object.assign({}, defaultProductCate);
        this.getSelectProductCateList();
        this.filterProductAttrList = [{
          value: []
        }];
      },
      removeFilterAttr(productAttributeId) {
        if (this.filterProductAttrList.length === 1) {
          this.$message({
            message: '至少要留一个',
            type: 'warning',
            duration: 1000
          });
          return;
        }
        var index = this.filterProductAttrList.indexOf(productAttributeId);
        if (index !== -1) {
          this.filterProductAttrList.splice(index, 1)
        }
      },
      handleAddFilterAttr() {
        if (this.filterProductAttrList.length === 3) {
          this.$message({
            message: '最多添加三个',
            type: 'warning',
            duration: 1000
          });
          return;
        }
        this.filterProductAttrList.push({
          value: null,
          key: Date.now()
        });
      }
    },
    filters: {
      filterLabelFilter(index) {
        if (index === 0) {
          return '筛选属性：';
        } else {
          return '';
        }
      }
    }
  }
</script>

<style scoped>

</style>
