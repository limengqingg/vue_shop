<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加分类按钮区域 -->
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
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        :show-index="true"
        index-text="#"
        :border="true"
        :show-row-hover="false"
      >
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen"
          ></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag
            type="success"
            size="mini"
            v-else-if="scope.row.cat_level === 1"
            >二级</el-tag
          >
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button size="mini" type="primary" icon="el-icon-edit"
              @click="showEditCateDialog(scope.row.cat_id)">编辑</el-button
          >
          <el-button size="mini" type="danger" icon="el-icon-delete"
            @click="removeCateById(scope.row.cat_id)">删除</el-button
          >
        </template>
      </tree-table>
      <!-- 分类列表区域 -->
      <!-- 索引列 -->
      <!-- <el-table :data="catelist" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="分类名称" prop="cat_name"></el-table-column>
        <el-table-column label="是否有效" prop="cat_pid"></el-table-column>
        <el-table-column label="排序" prop="cat_level"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template>
            <el-button size="mini" type="primary" icon="el-icon-edit"
              >编辑</el-button
            >
            <el-button size="mini" type="danger" icon="el-icon-delete"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table> -->
      <!-- 分页区域 -->
      <el-pagination
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        layout="total, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
    >
      <!-- 添加分类的表单 -->
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <!-- options用来指定数据源 -->
          <!-- props 用来指定配置对象 -->
          <el-cascader
          expand-trigger="hover"
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable
            change-on-select
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate"
          >确 定</el-button
        >
      </span>
    </el-dialog>

     <!-- 修改分类对话框 -->
    <el-dialog
      title="修改分类信息"
      :visible.sync="editCateEialogVisible"
      width="50%"
      @close="editCateDialogClosed"
    >
      <el-form
        :model="editCateForm"
        ref="editCateFormRef"
        label-width="50px"
      >
        <el-form-item label="分类名称">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateEialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCateInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //查询条件
      queryInfo: {
        type: 3,
        //当前页数
        pagenum: 1,
        //当前每页显示多少条数据
        pagesize: 5,
      },
      //商品分类的数据列表，默认为空
      catelist: [],
      total: 0,
      //为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name',
        },
        {
          label: '是否有效',
          //表示将当前列定义为模板列
          type: 'template',
          //表示当前这一列使用的模板名称
          template: 'isok',
        },
        {
          label: '排序',
          //表示将当前列定义为模板列
          type: 'template',
          //表示当前这一列使用的模板名称
          template: 'order',
        },
        {
          label: '操作',
          //表示将当前列定义为模板列
          type: 'template',
          //表示当前这一列使用的模板名称
          template: 'opt',
        },
      ],
      //控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      //添加分类的表单数据对象
      addCateForm: {
        //将要添加的分类的名称
        cat_name: '',
        //父级分类的id
        cat_pid: 0,
        //分类的等级，默认要添加的是1级分类
        cat_level: 0,
      },
      //添加分类表单的验证规则对象
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类的名称', trigger: 'blur' },
        ],
      },
      //父级分类的列表
      parentCateList: [],
      //指定级联选择器的配置对象
      cascaderProps:{
        value:'cat_id',
        label:'cat_name',
        children:'children'
      },
      //选中的父级分类的Id数组
      selectedKeys:[],
      //修改分类对话框
      editCateEialogVisible:false,
      editCateForm:{}
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //商品分类的数据列表
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo,
      })
      if (res.meta.status !== 200) {
        this.$message.error('获取商品分类列表失败')
      }
      console.log(res.data)
      this.total = res.data.total
      this.catelist = res.data.result
    },
    // 监听页码值改变的事件
    handleCurrentChange(newPage) {
      console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    //点击“添加分类”按钮，显示添加分类对话框
    showAddCateDialog() {
      //先获取父级分类的数据列表
      this.getParentCateList()
      this.addCateDialogVisible = true
    },

    //获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 },
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类的数据列表失败！')
      }
      console.log(res.data)
      this.parentCateList = res.data
    },
    //选择项发生变化触发这个函数
    parentCateChanged(){
      console.log(this.selectedKeys)
      //如果selectedKeys数组中的 length 大于0 ，证明选中的父级分类
      //反之，就说明没有选中任何父级分类
      if(this.selectedKeys.length > 0){
        //父级分类的Id
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        //为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
        return
      } else {
         //父级分类的Id
        this.addCateForm.cat_pid = 0
        //为当前分类的等级赋值
        this.addCateForm.cat_level = 0
      }

    },
    //点击按钮，添加新的分类
    addCate(){
      console.log(this.addCateForm)
       this.$refs.addCateFormRef.validate(async valid =>{
         if(!valid) return
         const {data:res} =  await this.$http.post('categories',this.addCateForm)
         if(res.meta.status !== 201){
           this.$message.error('添加新的分类失败！')
         }
         this.$message.success('添加新的分类成功！')
         this.getCateList()
         this.addCateDialogVisible = false
       })
    },
    //监听对话框的关闭事件，重置表单数据
    addCateDialogClosed(){
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    
    //展示编辑分类的对话框
    async showEditCateDialog(id){
      const { data: res } = await this.$http.get('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询分类信息失败')
      }

      this.editCateForm = res.data
      this.editCateEialogVisible = true
    },
    
    //修改分类信息
    async editCateInfo(){
      console.log('editCateForm----',this.editCateForm)
      const {data:res} = await this.$http.put('categories/'+this.editCateForm.cat_id,{cat_name:this.editCateForm.cat_name})
      if (res.meta.status !== 200) {
          return this.$message.error('更新分类信息失败')
        }
          //关闭对话框
        this.editCateEialogVisible = false
        //刷新数据列表
        this.getCateList()
        //提示修改成功
        this.$message.success('更新分类信息成功')
    },

    //监听修改分类对话框的关闭事件
    
    editCateDialogClosed(){
        //拿到表单的引用
      this.$refs.editCateFormRef.resetFields()
    },

    //删除removeCateById
    async removeCateById(id){
      const confirmResult = await this.$confirm('此操作将永久删除该分类,是否继续?','提示',{
        confirmButtonText:'确定',
        cancelButtonText:'取消',
        type:'warning'
      }).catch(err=>{
        return err
      })

      console.log(confirmResult)
      if(confirmResult !== 'confirm'){
        return this.$message.info('已取消删除')
      }
      //删除分类   delete请求
      const{data:res} = await this.$http.delete('categories/'+id)
      if(res.meta.status !== 200){
        return this.$message.error('删除分类失败')
      }
      this.$message.success('删除分类成功')
      this.getCateList()
    }
  },
}
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}

.el-cascader{
  width: 100%;
}


</style>