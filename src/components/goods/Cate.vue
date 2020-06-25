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
          <el-button @click="showAddCateDialog" type="primary"
            >添加分类</el-button
          >
        </el-col>
      </el-row>

      <!-- 表格 -->
      <tree-table
        class="treeTable"
        show-index
        border
        :selection-type="false"
        :expand-type="false"
        :data="cateList"
        :columns="columns"
      >
        <!-- 是否有效模板 -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen;"
          ></i>
          <i style="color: lightgreen;" class="el-icon-error" v-else></i>
        </template>

        <!-- 排序模板 -->
        <template v-slot:order="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag size="mini" v-if="scope.row.cat_level === 1" type="success"
            >二级</el-tag
          >
          <el-tag size="mini" v-if="scope.row.cat_level === 2" type="warning"
            >三级</el-tag
          >
        </template>

        <!-- 操作模板 -->
        <template v-slot:opt="scope">
          <el-button
            size="mini"
            type="primary"
            @click="showEditDialog(scope.row)"
            >编辑</el-button
          >
          <el-button
            size="mini"
            type="danger"
            @click="showDeleteDialog(scope.row.cat_id)"
            >删除</el-button
          >
        </template>
      </tree-table>

      <!-- 分页区域 -->
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
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 添加分类的表单 -->
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <!-- opintions用来指定数据源 -->
          <el-cascader
            clearable
            v-model="selectedKeys"
            :options="parentCateList"
            :props="{
              expandTrigger: 'hover',
              value: 'cat_id',
              label: 'cat_name',
              children: 'children',
              checkStrictly: true
            }"
            @change="parentCateChanged"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑分类的对话框 -->
    <el-dialog
      title="编辑分类"
      :visible.sync="editCateDialogVisible"
      width="50%"
    >
      <!-- 编辑的表单 -->
      <el-form :model="editCateForm" ref="editCateFormRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span class="dialog-footer">
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
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类的数据列表，默认为空
      cateList: [],
      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ],
      // 控制添加分类的对话框的显示与隐藏
      addCateDialogVisible: false,
      // 添加分类的表单数据对象:
      addCateForm: {
        // 将要添加的分类的名称
        cat_name: '',
        // 父级分类的ID
        cat_pid: 0,
        // 分类的等级，默认要添加的是一级分类
        cat_level: 0
      },
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
        children: 'children'
      },
      // 选中的父级分类的id数组
      selectedKeys: [],
      // 编辑的表单数据对象
      editCateForm: {
        cat_id: 0,
        cat_name: ''
      },
      editCateDialogVisible: false
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })

      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }

      this.cateList = res.data.result
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击按钮，展示添加分类的对话框
    showAddCateDialog() {
      // 先获取父级分类的数据列表
      this.getParentCateList()
      // 再展示对话框
      this.addCateDialogVisible = true
    },
    addDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    // 获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })

      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类数据失败！')
      }

      this.parentCateList = res.data
    },
    // 选择性发生变化触发这个函数
    parentCateChanged() {
      // selectedKeys数组的length为0代表没选中任何东西
      // length为1代表只选中了第一级，length为2代表选中了第二级
      if (this.selectedKeys.length > 0) {
        // 父级分类的ID
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        // 父级分类的ID
        this.addCateForm.cat_pid = 0
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )

        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }

        this.$message.success('添加分类成功！')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    async showEditDialog(row) {
      this.editCateDialogVisible = true
      this.editCateForm.cat_name = row.cat_name
      this.editCateForm.cat_id = row.cat_id
    },
    async showDeleteDialog(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该商品分类，是否继续？',
        '提示',
        { confirmButtonText: '确定', cancelButtonText: '取消', type: 'warning' }
      ).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消本次删除')
      }

      const { data: res } = await this.$http.delete(`categories/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除商品分类失败！')
      }

      this.$message.success('删除商品分类成功！')
      this.getCateList()
    },
    // 编辑商品分类
    async editCate() {
      const { data: res } = await this.$http.put(
        `categories/${this.editCateForm.cat_id}`,
        {
          cat_name: this.editCateForm.cat_name
        }
      )

      if (res.meta.status !== 200) {
        return this.$message.error('修改商品分类失败！')
      }

      this.$message.success('修改商品分类成功！')
      this.getCateList()
      this.editCateDialogVisible = false
    }
  }
}
</script>

<style lang="scss" scoped>
.treeTable {
  margin-top: 15px;
}

.el-cascader {
  width: 100%;
}
</style>
