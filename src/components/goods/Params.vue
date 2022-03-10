<template>
  <div>
    <!-- 面包屑导航区-->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片试图 -->
    <el-card>
      <!-- 提醒区 -->
      <el-alert
        :closable="false"
        show-icon
        title="注意：只允许为第三级分类选择参数"
        type="warning"
      >
      </el-alert>
      <!-- 选择商品分类区 -->
      <el-row class="cat_top">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 选择商品分类的级联框 -->
          <el-cascader
            :options="cateList"
            :props="cascaderProps"
            v-model="selectedKeys"
            @change="cateChanged"
            :clearable="true"
          >
          </el-cascader>
        </el-col>
      </el-row>
      <!-- tab 页签区-->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 添加动态参数的面板 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnVisible"
            @click="addDialogVisible = true"
            >添加参数</el-button
          >
          <!-- 动态参数表格 -->
          <el-table border="" stripe="" :data="manyTableData">
            <!-- 展开行 -->
            <el-table-column type="expand" label="展开">
              <template slot-scope="scope">
                <!-- 循环渲染便签 -->
                <el-tag
                  v-for="(item, index) in scope.row.attr_vals"
                  :key="index"
                  :closable="true"
                  @close="handleClose(scope.row, index)"
                  >{{ item }}</el-tag
                >
                <!-- 输入文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >这是添加的按钮</el-button
                >
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index" label="序号"></el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  size="mini"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                  >编辑</el-button
                >
                <el-button
                  type="danger"
                  size="mini"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态属性的面板 -->
        <el-tab-pane label="静态属性" name="only" :disabled="isBtnVisible">
          <el-button type="primary" size="mini" @click="addDialogVisible = true"
            >添加属性</el-button
          >

          <!-- 静态属性表格 -->
          <el-table border="" stripe="" :data="onlyTableData">
            <!-- 展开行 -->
            <el-table-column type="expand" label="展开">
              <template slot-scope="scope">
                <!-- 循环渲染便签 -->
                <el-tag
                  v-for="(item, index) in scope.row.attr_vals"
                  :key="index"
                  :closable="true"
                  @close="handleClose(scope.row, index)"
                  >{{ item }}</el-tag
                >
                <!-- 输入文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >这是添加的按钮</el-button
                >
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index" label="序号"></el-table-column>
            <el-table-column
              label="属性名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  size="mini"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                  >编辑</el-button
                >
                <el-button
                  type="danger"
                  size="mini"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数的对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改对话框 -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 商品分类列表
      cateList: [],
      // 级联选择框配置选项
      cascaderProps: {
        value: 'cat_id',
        // 看到的属性
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover'
      },
      //   级联选择框双向绑定到的数组
      selectedKeys: [],
      // 被激活的标签页的名称
      activeName: 'many',
      //   动态参数表单内容
      manyTableData: [],
      //   静态属性表格内容
      onlyTableData: [],
      // 控制添加对话框的显示与隐藏
      addDialogVisible: false,
      // 添加属性表单
      addForm: {
        attr_name: ''
      },
      // 添加属性表单验证规则对象
      addFormRules: {
        attr_name: [
          {
            required: true,
            message: '请输入参数名称',
            trigger: 'blur'
          }
        ]
      },
      // 控制修改对话框的显示与隐藏
      editDialogVisible: false,
      // 修改参数的数据对象
      editForm: [],
      // 修改属性表单验证规则对象
      editFormRules: {
        attr_name: [
          {
            required: true,
            message: '请输入参数名称',
            trigger: 'blur'
          }
        ]
      }
      //   控制文本框与按钮的切换显示  所有for循环都一样了，这样是错的
      // inputVisible: false,
      //   这是文本框输入的内容
      // inputValue: ''
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败')
      }
      this.cateList = res.data
      console.log(this.cateList)
    },
    // 级联选择框选择项改变，会触发这函数
    cateChanged() {
      this.getParamsData()
      console.log(this.activeName)
    },
    // tab页签点击事件处理函数
    handleTabClick() {
      this.getParamsData()
      console.log(this.activeName)
      //  console.log(this.onlyTableData)
    },
    // 获取选中ID的参数
    async getParamsData() {
      // 选中的不是三级分类
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      // 选中的是三级分类
      //   根据所选择的ID，和当前所处面板，获取对应的参数
      console.log(this.selectedKeys)
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        { params: { sel: this.activeName } }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败')
      }
      //   将拿到的字符串编程数组
      res.data.forEach((item) => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 控制文本框的显示与隐藏
        item.inputVisible = false
        // 文本框输入的值
        item.inputValue = ''
      })

      console.log(res.data)
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      }
      this.onlyTableData = res.data
    },
    // 监听对话框的关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 点击添加按钮，添加参数
    addParams() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          { attr_name: this.addForm.attr_name, attr_sel: this.activeName }
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败')
        }
        this.$message.success('添加参数成功')
        this.addDialogVisible = false
        this.getParamsData()
      })
    },
    // 点击按钮，展示修改对话框
    async showEditDialog(attrId) {
      // 查询当前参数的信息
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${attrId}`,
        { params: { attr_sel: this.activeName } }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数失败')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 监听修改参数对话框的关闭事件
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 点击按钮，修改参数
    editParams() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return
        // 预校验成功，提交修改请求
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          { attr_name: this.editForm.attr_name, attr_sel: this.activeName }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改参数失败')
        }
        this.$message.success('修改参数成功')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    async removeParams(attrId) {
      // 删除提示弹框
      const confirmResult = await this.$confirm(
        '此操作将永久删除该参数, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch((err) => err)
      //   用户取消删除
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除参数操作')
      }
      // 删除业务逻辑
      const { data: res } = await this.$http.delete(
        `categories/${this.cateId}/attributes/${attrId}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败')
      }
      this.$message.success('删除参数成功')
      this.getParamsData()
    },
    // 文本框失去焦点或者按下enter按键触发

    handleInputConfirm(row) {
      // 检查输入内容的合法性
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        //   让文本框变成按钮
        row.inputVisible = false
        return
      }
      // 输入合法内容
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      // 发起请求
      this.saveAttrVals(row)
    },
    // 点击按钮展示文本输入框
    showInput(row) {
      row.inputVisible = true
      //   让文本框自动获取焦点
      //   $nextTick方法的作用，当页面上的元素被重新渲染之后，才执行回掉函数
      this.$nextTick((_) => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    async saveAttrVals(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('修改参数项失败')
      }
      this.$message.success('修改参数项成功')
    },
    // 监听标签的close事件，删除参数
    handleClose(row, index) {
      row.attr_vals.splice(index, 1)
      this.saveAttrVals(row)
    }
  },
  computed: {
    // 如果希望按钮被禁用，就返回true,否则返回false
    isBtnVisible() {
      if (this.selectedKeys.length !== 3) {
        return true
      }
      return false
    },
    // 当前选中三级分类的ID
    cateId() {
      if (this.selectedKeys.length === 3) {
        return this.selectedKeys[2]
      }
      return null
    },
    // 添加框的文本
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  }
}
</script>
<style lang="less" scoped>
.cat_top {
  margin-top: 10px;
}
.el-tag {
  margin: 5px;
}
.input-new-tag {
  width: 150px;
}
</style>
