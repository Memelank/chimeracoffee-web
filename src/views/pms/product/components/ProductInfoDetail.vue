<template>
  <div style="margin-top: 50px">
    <el-form :model="value" :rules="rules" ref="productInfoForm" label-width="120px" class="form-inner-container" size="small">
      <el-form-item label="商品分类：" prop="productCategoryId">
        <el-cascader
          v-model="selectProductCateValue"
          :options="productCateOptions">
        </el-cascader>
      </el-form-item>
      <el-form-item label="商品名称：" prop="name">
        <el-input v-model="value.name"></el-input>
      </el-form-item>
      <!-- TODO  修改为简介-->
      <el-form-item label="副标题：" prop="subTitle">
        <el-input v-model="value.subTitle"></el-input>
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
<!--      <el-form-item label="市场价：">-->
<!--        <el-input v-model="value.originalPrice"></el-input>-->
<!--      </el-form-item>-->
<!--      <el-form-item label="商品库存：">-->
<!--        <el-input v-model="value.stock"></el-input>-->
<!--      </el-form-item>-->
<!--      <el-form-item label="计量单位：">-->
<!--        <el-input v-model="value.unit"></el-input>-->
<!--      </el-form-item>-->
<!--      <el-form-item label="商品重量：">-->
<!--        <el-input v-model="value.weight" style="width: 300px"></el-input>-->
<!--        <span style="margin-left: 20px">克</span>-->
<!--      </el-form-item>-->
      <el-form-item label="排序">
        <el-input v-model="value.sort"></el-input>
      </el-form-item>

      <!-- 动态生成属性选择行 -->
      <div v-for="(attrRow, rowIndex) in selectedAttributes" :key="rowIndex">
        <el-form-item :label="'属性类型：' + (rowIndex + 1)">
          <el-select v-model="attrRow.attribute" placeholder="请选择属性类型" @change="onAttributeChange(rowIndex)">
            <el-option
              v-for="item in productAttributeCategoryOptions"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
          <el-button type="danger" @click="removeAttributeRow(rowIndex)">删除属性</el-button>
        </el-form-item>

        <!-- 动态生成属性值选择 -->
        <el-form-item v-if="attrRow.options.length > 0" label="属性值：">
          <el-select v-model="attrRow.selectedOptions" placeholder="请选择属性值" multiple>
            <el-option
              v-for="option in attrRow.options"
              :key="option"
              :label="option"
              :value="option">
            </el-option>
          </el-select>
<!--          <el-button-->
<!--            v-for="(option, index) in attrRow.selectedOptions"-->
<!--            :key="index"-->
<!--            @click="removeOption(rowIndex, option)">-->
<!--            {{ option }} 删除-->
<!--          </el-button>-->
        </el-form-item>
      </div>

      <!-- 添加属性行按钮 -->
      <el-button @click="addAttributeRow">添加属性</el-button>


<!--       图片上传-->
      <el-form-item label="上传图片：">
        <el-upload
          class="upload-demo"
          action="https://jsonplaceholder.typicode.com/posts/"
        list-type="picture-card"
        :on-preview="handlePictureCardPreview"
        :on-remove="handleRemove"
        :on-success="handleSuccess"
        :file-list="fileList">
        <i class="el-icon-plus"></i>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible">
          <img width="100%" :src="dialogImageUrl" alt="">
        </el-dialog>
      </el-form-item>


      <el-form-item style="text-align: center">
        <el-button type="primary" size="medium" @click="handleNext('productInfoForm')">下一步，填写商品促销</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
  import {fetchListWithChildren} from '@/api/productCate'
  // import {fetchList as fetchBrandList} from '@/api/brand'
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
        hasEditCreated:false,
        //选中商品分类的值
        selectProductCateValue: [],
        productCateOptions: [],
        // brandOptions: [],
        rules: {
          name: [
            {required: true, message: '请输入商品名称', trigger: 'blur'},
            {min: 2, max: 140, message: '长度在 2 到 140 个字符', trigger: 'blur'}
          ],
          subTitle: [{required: true, message: '请输入商品副标题', trigger: 'blur'}],
          productCategoryId: [{required: true, message: '请选择商品分类', trigger: 'blur'}],
          // brandId: [{required: true, message: '请选择商品品牌', trigger: 'blur'}],
          description: [{required: true, message: '请输入商品介绍', trigger: 'blur'}],
          requiredProp: [{required: true, message: '该项为必填项', trigger: 'blur'}],



        },

        productAttributeCategoryOptions: [
          { value: 'temperature', label: '温度' },
          { value: 'sweetness', label: '甜度' },
        ],
        attributeValues: {
          temperature: ['去冰', '常温'],
          sweetness: ['全糖', '半糖'],
        },
        // 存储用户动态添加的属性行
        selectedAttributes: [],

        // 上传图片
        form: {
          imageUrl: '' // 存储图片URL
        },
        dialogImageUrl: '', // 预览图片的URL
        dialogVisible: false, // 控制图片预览弹窗的显示
        fileList: [] // 存储上传的文件列表
      };
    },
    created() {
      this.getProductCateList();
      // this.getBrandList();
    },
    computed:{
      //商品的编号
      productId(){
        return this.value.id;
      }
    },
    watch: {
      productId:function(newValue){
        if(!this.isEdit)return;
        if(this.hasEditCreated)return;
        if(newValue===undefined||newValue==null||newValue===0)return;
        this.handleEditCreated();
      },
      selectProductCateValue: function (newValue) {
        if (newValue != null && newValue.length === 2) {
          this.value.productCategoryId = newValue[1];
          this.value.productCategoryName= this.getCateNameById(this.value.productCategoryId);
        } else {
          this.value.productCategoryId = null;
          this.value.productCategoryName=null;
        }
      }
    },
    methods: {
      //处理编辑逻辑
      handleEditCreated(){
        if(this.value.productCategoryId!=null){
          this.selectProductCateValue.push(this.value.cateParentId);
          this.selectProductCateValue.push(this.value.productCategoryId);
        }
        this.hasEditCreated=true;
      },
      getProductCateList() {
        fetchListWithChildren().then(response => {
          let list = response.data;
          this.productCateOptions = [];
          for (let i = 0; i < list.length; i++) {
            let children = [];
            if (list[i].children != null && list[i].children.length > 0) {
              for (let j = 0; j < list[i].children.length; j++) {
                children.push({label: list[i].children[j].name, value: list[i].children[j].id});
              }
            }
            this.productCateOptions.push({label: list[i].name, value: list[i].id, children: children});
          }
        });
      },
      // getBrandList() {
      //   fetchBrandList({pageNum: 1, pageSize: 100}).then(response => {
      //     this.brandOptions = [];
      //     let brandList = response.data.list;
      //     for (let i = 0; i < brandList.length; i++) {
      //       this.brandOptions.push({label: brandList[i].name, value: brandList[i].id});
      //     }
      //   });
      // },
      getCateNameById(id){
        let name=null;
        for(let i=0;i<this.productCateOptions.length;i++){
          for(let j=0;j<this.productCateOptions[i].children.length;j++){
            if(this.productCateOptions[i].children[j].value===id){
              name=this.productCateOptions[i].children[j].label;
              return name;
            }
          }
        }
        return name;
      },
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
      // handleBrandChange(val) {
      //   let brandName = '';
      //   for (let i = 0; i < this.brandOptions.length; i++) {
      //     if (this.brandOptions[i].value === val) {
      //       brandName = this.brandOptions[i].label;
      //       break;
      //     }
      //   }
      //   this.value.brandName = brandName;
      // }

      // 动态添加属性行
      addAttributeRow() {
        this.selectedAttributes.push({
          attribute: '',
          options: [],
          selectedOptions: [],
        });
      },
      // 当属性类型改变时，动态更新可选值
      onAttributeChange(rowIndex) {
        const selectedAttribute = this.selectedAttributes[rowIndex].attribute;
        this.selectedAttributes[rowIndex].options = this.attributeValues[selectedAttribute] || [];
        this.selectedAttributes[rowIndex].selectedOptions = []; // 重置选项
      },
      // 删除属性行
      removeAttributeRow(rowIndex) {
        this.selectedAttributes.splice(rowIndex, 1);
      },
      // 删除某个属性值
      removeOption(rowIndex, option) {
        const index = this.selectedAttributes[rowIndex].selectedOptions.indexOf(option);
        if (index !== -1) {
          this.selectedAttributes[rowIndex].selectedOptions.splice(index, 1);
        }
      },


      // 处理图片预览
      handlePictureCardPreview(file) {
        this.dialogImageUrl = file.url || file.raw;
        this.dialogVisible = true;
      },
      // 处理删除图片
      handleRemove(file, fileList) {
        console.log(file, fileList);
      },
      // 图片上传成功后处理
      handleSuccess(response, file, fileList) {
        this.form.imageUrl = file.url; // 这里假设服务器返回了图片的 URL
        console.log('图片上传成功', response);
      },
    }
  }
</script>

<style scoped>
.upload-demo .el-upload {
  width: 100px;
  height: 100px;
  border: 1px dashed #d9d9d9;
  border-radius: 4px;
  position: relative;
  overflow: hidden;
}

.upload-demo .el-upload:hover {
  border-color: #409EFF;
}
</style>
