<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCatDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <el-row>
        <!-- 表格 -->
        <tree-table :data="cateList" :columns="columns" :selection-type="false"
        :expand-type="false" :show-index="true" index-text="#" border
        :show-row-hover="true"
        class="treeTable">
        <!-- 是否有效 -->
          <template slot="isok" slot-scope="scope">
            <i class='el-icon-success' v-if="scope.row.cat_deleted === false"
            style="color: lightgreen;"></i>
            <i class='el-icon-error' v-else
            style="color: red;"></i>
          </template>
          <!-- 排序 -->
          <template slot="order" slot-scope="scope">
            <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
            <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
            <el-tag type="warning" size="mini" v-else>三级</el-tag>
          </template>
          <!-- 操作 -->
          <template slot="opt">
            <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
          </template>
        </tree-table>
        <!-- 分页区域 -->
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="queryInfo.pagenum"
          :page-size="queryInfo.pagesize"
          layout="total, prev, pager, next, jumper"
          :total="total">
        </el-pagination>
      </el-row>
    </el-card>
    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed">
      <!-- 添加分类的表单 -->
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader
            expand-trigger="hover"
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable>
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类的数据列表,默认为空
      cateList: [],
      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [{
        label: '分类名称',
        prop: 'cat_name'
      },
      {
        label: '是否有效',
        // 表示当前列定义为模板列
        type: 'template',
        // 表示当前这一列使用的模板名称
        template: 'isok'
      },
      {
        label: '排序',
        // 表示当前列定义为模板列
        type: 'template',
        // 表示当前这一列使用的模板名称
        template: 'order'
      },
      {
        label: '操作',
        // 表示当前列定义为模板列
        type: 'template',
        // 表示当前这一列使用的模板名称
        template: 'opt'
      }],
      // 控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      // 添加分类表单数据对象
      addCateForm: {
        // 将要添加的分类的名称
        cat_name: '',
        // 父级分类的id
        cat_pid: 0,
        // 分类的等级默认要添加的是1级分类
        cat_level: 0
      },
      // 表单的验证规则对象
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 父级分类的列表
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true
      },
      // 选择的父级分类ID数组
      selectedKeys: []
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    // 获取商品分类列表
    async getCateList () {
      const { data: res } = await this.$http.get('/categories', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取商品分类失败！')
      console.log(res.data)
      // 把数据列表,赋值给cateList
      this.cateList = res.data.result
      // 为总数据条数赋值
      this.total = res.data.total
    },
    // 监听pagesize改变事件
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pagenum的改变
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    showAddCatDialog () {
      // 先获取父级分类数据列表
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类失败!')
      }
      this.parentCateList = res.data
      console.log(res.data)
    },
    // 选择项发生变化触发
    parentCateChanged () {
      console.log(this.selectedKeys)
      // 如果selectedKeys数组中的length大于0,证明选中了父级分类
      // 反之,则说明没有选中任何分类
      if (this.selectedKeys.length > 0) {
        // 父级分类的ID
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 当前分类等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        // 父级分类的ID
        this.addCateForm.cat_pid = 0
        // 当前分类等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    // 点击按钮,添加新分类
    addCate () {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return false
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }
        this.$message.success('添加分类成功!')
        this.getCateList()
        this.addCateDialogVisible = false
      })
      console.log(this.addCateForm)
    },
    // 监听对话框的关闭事件,重置表单数据
    addCateDialogClosed () {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    }
  }
}
</script>

<style lang="less" scoped>
  .treeTable {
    margin-top: 15px;
  }
  .el-cascader {
    width: 100%;
  }
</style>
