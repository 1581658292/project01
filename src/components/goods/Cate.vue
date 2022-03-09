<template>
  <div>
    <!-- 面包屑导航区-->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card>
      <!-- 添加分类按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog"
            >添加分类</el-button
          >
        </el-col>
      </el-row>
      <!-- 表格 -->
      <tree-table
        class="treeTable"
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        border
        :show-row-hover="false"
      >
        <!-- 是否有效 -->
        <template slot="isOK" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen"
          ></i>
          <i class="el-icon-error" v-else></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0" size="mini">一级</el-tag>
          <el-tag
            type="warning"
            v-else-if="scope.row.cat_level === 1"
            size="mini"
            >二级</el-tag
          >
          <el-tag type="success" v-else size="mini">三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt">
          <el-button type="primary" icon="el-icon-edit" size="mini"
            >编辑</el-button
          >
          <el-button type="danger" icon="el-icon-delete" size="mini"
            >删除</el-button
          >
        </template>
      </tree-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCatedialogVisible"
      width="50%"
      @close="closeAddCateDialog"
    >
      <!-- 添加分类的表单 -->
      <el-form
        ref="addCateFormRef"
        :model="addCateForm"
        :rules="addCateFormRules"
        width="100px"
      >
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <el-cascader
            :options="parentCateList"
            :props="cascaderProps"
            v-model="selectedKeys"
            @change="parentCateChanged"
            :clearable="true"
          >
          </el-cascader>
        </el-form-item>
      </el-form>
      <!-- 添加分类对话框的底部 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCatedialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 商品分类的数据列表，默认为空
      queryInfo: {
        type: [3],
        pagenum: 1,
        pagesize: 5
      },
      // 获取到的商品分类列表
      cateList: [],
      //   获取到的总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 表示将当前列渲染为模板列
          type: 'template',
          // 表示当前这一列使用模板的名称
          template: 'isOK'
        },
        {
          label: '排序',
          // 表示将当前列渲染为模板列
          type: 'template',
          // 表示当前这一列使用模板的名称
          template: 'order'
        },
        {
          label: '操作',
          // 表示将当前列渲染为模板列
          type: 'template',
          // 表示当前这一列使用模板的名称
          template: 'opt'
        }
      ],
      // 控制添加分类对话框的显示与隐藏
      addCatedialogVisible: false,
      // 添加分类的表单数据对象
      addCateForm: {
        // 将要添加的分类的名称
        cat_name: '',
        // 父级分类的ID
        // 添加的是一级分类，他的父级就是0.其他的选中谁，谁的id就是他的父级id
        cat_pid: 0,
        // 添加的分类的等级，默认0是添加一级分类
        cat_level: 0
      },
      // 添加分类的表单数据验证规则
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
          //   { min: 3, max: 10, message: '长度在 3 到 10个字符', trigger: 'blur' }
        ]
      },
      // 获取到的父级分类列表
      parentCateList: [],
      // 指定cascader的配置对象
      cascaderProps: {
        //
        value: 'cat_id',
        // 看到的属性
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover',
        checkStrictly: 'true'
      },
      // 选中的父级分类的ID数组
      selectedKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //   获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类列表数据失败')
      }
      this.cateList = res.data.result
      this.total = res.data.total
      console.log(this.cateList)
    },
    // 监听pagesize的事件
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pagenum的改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击添加分类按钮，展示添加分类的对话框
    showAddCateDialog() {
      // 先获取父级分类数据
      this.getParentCate()
      //   再显示对话框
      this.addCatedialogVisible = true
    },
    // 获取父级分类的列表
    async getParentCate() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类数据失败')
      }
      this.parentCateList = res.data
      console.log(this.parentCateList)
    },
    // 选择项发生改变，触发这个函数
    parentCateChanged() {
      console.log(this.selectedKeys)

      if (this.selectedKeys.length > 0) {
        // 如果length大于0，证明选中父级
        this.addCateForm.cat_pid =
          this.selectedKeys[this.selectedKeys.length - 1]
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        //   如果length等于0，证明没有选中父级
        this.addCateForm.cat_pid = 0
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    // 点击确定，添加新的分类
    addCate() {
      console.log(this.addCateForm)
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加新分类失败')
        }
        this.$message.success('添加新分类成功')
        this.getCateList()
        this.addCatedialogVisible = false
      })
    },
    // 添加对话框关闭，清空表单数据
    closeAddCateDialog() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []

      this.addCateForm.cat_pid = 0

      this.addCateForm.cat_level = 0
    }
  }
}
</script>
<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
</style>
