<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>

      <!-- 表格 -->
      <el-table :data="cateList" style="width: 100%;margin-bottom: 20px;" row-key="cat_id" border :default-expand-all=false :tree-props="{children: 'children'}">
        <el-table-column prop="cat_name" label="分类名称" sortable>
        </el-table-column>
        <el-table-column label="是否有效" sortable>
          <template v-slot="scope">
            <el-switch v-model="scope.row.cat_deleted" disabled>
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column prop="cat_level" label="排序" sortable>
          <template v-slot="scope">
            <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
            <el-tag v-if="scope.row.cat_level === 1" type="success">二级</el-tag>
            <el-tag v-if="scope.row.cat_level === 2" type="warning">三级</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="cat_name" label="操作" sortable width="200px">
          <template v-slot="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showCateDialog(scope.row.cat_id)">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="deleteCate(scope.row.cat_id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="querInfo.pagenum" :page-sizes="[3, 5, 10, 15]" :page-size="querInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
      <!-- 添加分类的表单 -->
      <el-form ref="addCateFormRef" :model="addCateForm" label-width="100px" :rules="addCateFormRules">
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <el-cascader v-model="selectedKeys" :options="parentCateList" :props="cascaderProps" @change="parentCateChanged" clearable change-on-select></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑分类对话框 -->
    <el-dialog title="修改分类" :visible.sync="editCateDialogVisible" width="50%" @close="editCateDialogClosed">
      <!-- 修改分类的表单 -->
      <el-form ref="editCateFormRef" :model="editCateForm" label-width="100px" :rules="editCateFormRules">
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 查询参数
      querInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类的数据列表
      cateList: [],
      // 总数据条数
      total: 0,
      // 控制添加分类对话框的显示隐藏
      addCateDialogVisible: false,
      // 添加分类表单的数据对象
      addCateForm: {
        // 分类名称
        cat_name: '',
        // 父级分类的Id
        cat_pid: 0,
        // 添加分类的等级
        cat_level: 0
      },
      // 添加分类表单的验证规则对象
      addCateFormRules: {
        cat_name: [{ required: true, message: '请输入分类名称', trigger: 'blur' }]
      },
      // 父级分类的列表
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中的父级分类Id数组
      selectedKeys: [],
      // 修改分类的显示隐藏对话框
      editCateDialogVisible: false,
      // 查询修改分类的表单数据
      editCateForm: {},
      // 修改分类表单的验证规则对象
      editCateFormRules: {
        cat_name: [{ required: true, message: '请输入分类名称', trigger: 'blur' }]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories', { params: this.querInfo })
      if (res.meta.status !== 200) return this.$Message.error('获取商品分类失败！')
      this.cateList = res.data.result
      this.total = res.data.total
    },
    // 监听 pagesize 改变
    handleSizeChange(newSize) {
      this.querInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听 pagenum 改变
    handleCurrentChange(newPage) {
      this.querInfo.pagenum = newPage
      this.getCateList()
    },
    // 添加分类按钮
    showAddCateDialog() {
      // 获取父级分类数据列表
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) return this.$Message.error('获取父级分类数据失败！')
      this.parentCateList = res.data
    },
    // 选择项发生变化触发
    parentCateChanged() {
      // 如果 selectedKeys 数组中的 length 大于 0，证明选中了父级分类
      // 反之，就说明没有选中任何父级分类
      if (this.selectedKeys.length > 0) {
        // 父级分类的Id
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
        return
      } else {
        // 父级分类的Id
        this.addCateForm.cat_pid = 0
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    // 添加分类并提交
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) return this.$Message.error('添加分类失败！')
        this.$Message.success('添加分类成功！')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    // 关闭添加分类关闭事件
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.cat_level = 0
      this.cat_pid = 0
    },
    // 编辑分类名称
    async showCateDialog(id) {
      const { data: res } = await this.$http.get('categories/' + id)
      if (res.meta.status !== 200) return this.$Message.error('查询分类名称失败！')
      this.editCateForm = res.data
      this.editCateDialogVisible = true
    },
    // 编辑对话框关闭事件
    editCateDialogClosed() {
      this.$refs.editCateFormRef.resetFields()
    },
    // 编辑分类名称并提交
    editCate() {
      this.$refs.editCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('categories/' + this.editCateForm.cat_id, { cat_name: this.editCateForm.cat_name })
        if (res.meta.status !== 200) return this.$Message.error('更新分类名称失败！')
        this.$Message.success('更新分类名称成功！')
        this.getCateList()
        this.editCateDialogVisible = false
      })
    },
    // 删除分类按钮
    async deleteCate(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if(confirmResult !== 'confirm') return this.$Message.info('取消删除！')
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) return this.$Message.error('删除分类失败！')
      this.$Message.success('删除成功！')
      this.getCateList()
    }
  }
}
</script>

<style lang="less" scoped>
</style>