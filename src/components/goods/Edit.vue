<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>编辑商品</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 提示区域 -->
      <el-alert title="编辑商品信息" type="info" center show-icon :closable="false">
      </el-alert>
      <!-- 步骤条区域 -->
      <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>

      <!-- tab区域 -->
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" label-position="top">
        <el-tabs v-model="activeIndex" :tab-position="'left'" :before-leave="beforeTabLeave" @tab-click="tabClicked">
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="editForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="editForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="editForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="editForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader :options="cateList" :props="cateProps" v-model="editForm.goods_cat" expand-trigger='hover' @change="handleChange" style="width: 250px"></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
              <!-- 复选框 -->
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox :label="cb" v-for="(cb, i) in item.attr_vals" :key="i" border=""></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <!-- action 表示图片要上传到的后台api地址 -->
            <el-upload :action="uploadURL" :on-preview="handlePreview" :on-remove="handleRemove" list-type="picture" :headers="headerObj" :on-success="handleSuccess">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <!-- 富文本编辑器组件 -->
            <quill-editor v-model="this.editForm.goods_introduce"></quill-editor>
            <el-button type="primary" class="btnEdit" @click="edit">提交商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>

    <!-- 图片预览 -->
    <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
      <img :src="previewPath" class="previewImg">
    </el-dialog>
  </div>
</template>

<script>
import bus from '../EventBus'
// 深拷贝
import _ from 'lodash'

export default {
  data() {
    return {
      activeIndex: '0',
      // 编辑商品的表单数据对象
      editForm: {
        // 商品所属的分类数组
        goods_cat: [],
        // 商品详情描述
        goods_introduce: ''
      },
      // 商品图片
      pics: [],
      // 商品的参数（数组）
      attrs: [],
      // 编辑商品表单的验证规则
      editFormRules: {
        goods_name: [{ required: true, message: '请输入商品名称', trigger: 'blur' }],
        goods_price: [{ required: true, message: '请输入商品价格', trigger: 'blur' }],
        goods_weight: [{ required: true, message: '请输入商品重量', trigger: 'blur' }],
        goods_number: [{ required: true, message: '请输入商品数量', trigger: 'blur' }],
        goods_cat: [{ required: true, message: '请选择商品分类', trigger: 'blur' }]
      },
      // 商品分类列表数据
      cateList: [],
      cateProps: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      // 商品参数列表数据
      manyTableData: [],
      // 商品属性列表数据
      onlyTableData: [],
      // 上传图片的URL地址
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      // 图片上传组件的headers请求头
      headerObj: {
        Authorization: sessionStorage.getItem('token')
      },
      // 预览图片的路径
      previewPath: '',
      // 显示隐藏预览图片对话框
      previewVisible: false
    }
  },
  created() {
    this.getCateList()
    // 接收列表传来的参数
    bus.$on('editInfo', val => {
      this.editForm = {
        goods_id: val.goods_id,
        goods_name: val.goods_name,
        goods_price: val.goods_price,
        goods_number: val.goods_number,
        goods_weight: val.goods_weight
      }
      console.log(this.editForm)
    })
  },
  methods: {
    // 获取所有商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$Message.error('获取商品分类失败！')
      this.cateList = res.data
    },
    // 级联选择器
    handleChange(value) {
      this.editForm.goods_cat = value
      if (this.editForm.goods_cat.length !== 3) {
        this.editForm.goods_cat = []
      }
    },
    // 切换标签判断是否选中级联  activeName 即将进入的标签名
    beforeTabLeave(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.editForm.goods_cat.length !== 3) {
        this.$Message.error('请选择商品分类！')
        return false
      }
    },
    // 切换标签时触发
    async tabClicked() {
      // 证明访问的是商品参数面板
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'many' } })
        if (res.meta.status !== 200) return this.$Message.error('获取商品参数失败！')

        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(',')
        })
        this.manyTableData = res.data
      } else if (this.activeIndex === '2') {
        // 访问商品属性面板
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'only' } })
        if (res.meta.status !== 200) return this.$Message.error('获取商品属性失败！')
        this.onlyTableData = res.data
      }
    },
    // 处理图片预览效果
    handlePreview(file) {
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    // 删除图片
    handleRemove(file) {
      // 1. 获取将要删除的图片临时路径
      const filePath = file.response.data.tem_path
      // 2. 从 pics 数组中，找到这个图片对应的索引值
      const i = this.pics.findIndex(x => x.pic === filePath)
      // 3. 调用数组的 splice 方法，把图片信息对象，从pics数组中移除
      this.pics.splice(i, 1)
    },
    // 图片上传成功的事件
    handleSuccess(response) {
      // console.log(response)
      // 1. 拼接得到一个图片信息对象
      const picInfo = { pic: response.data.tmp_path }
      // 2. 将图片信息对象 push 到pics数组中
      this.pics.push(picInfo)
    },
    // 提交商品
    edit() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return this.$Message.error('请填写必要的表单项！')
        // 用深拷贝 把表单中的goods_cat数组转换为字符串再提交
        // lodash包  cloneDeep(obj)
        const form = _.cloneDeep(this.editForm)
        form.goods_cat = form.goods_cat.join(',')
        // 处理商品参数
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(',')
          }
          this.attrs.push(newInfo)
        })
        // 处理商品属性
        this.onlyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          this.attrs.push(newInfo)
        })
        form.attrs = this.attrs
        // 发请求提交商品，商品名称必须唯一
        const { data: res } = await this.$http.put(`goods/${this.editForm.goods_id}`, form)
        // if (res.meta.status !== 201) return this.$Message.error('编辑商品失败！')
        // this.$Message.success('编辑商品成功！')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    // 级联三级分类Id
    cateId() {
      if (this.editForm.goods_cat.length === 3) {
        return this.editForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 10px 0 0;
}

.previewImg {
  width: 100%;
}

.btnEdit {
  margin-top: 15px;
}
</style>