<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary"   @click="addRoleDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <!-- i1 === 0 ? 'bdtop' :''  索引为0 就加一个样式 bdtop-->
            <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' :'',,'vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag   closable 
                 @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
               <el-col :span="19">
                 <!-- 通过 for 循环 嵌套渲染二级权限 -->
                 <el-row :class="[i2==='0'?'':'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                   <el-col :span="6">
                     <el-tag   closable type="success"
                      @close="removeRightById(scope.row,item2.id)"> {{item2.authName}}</el-tag>
                     <i class="el-icon-caret-right"></i>
                   </el-col>
                   <el-col :span="18">
                     <el-tag   closable type="warning" :class="[i3 === '0' ? '':'bdtop']" 
                     v-for="(item3,i3) in item2.children" :key="item3.id"
                     @close="removeRightById(scope.row,item3.id)">{{item3.authName}}</el-tag>
                   </el-col>
                 </el-row>
               </el-col>
            </el-row>
            <!-- <pre>
              {{scope.row}}
            </pre> -->
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showEditDialog(scope.row.id)"
              >编辑</el-button
            >
            <el-button size="mini" type="danger" icon="el-icon-delete"
              @click="removeRoleById(scope.row.id)"
              >删除</el-button
            >
            <el-button size="mini" type="warning" icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)">分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 修改角色区域 -->
    <el-dialog
      title="修改角色信息"
      :visible.sync="editRoleEialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <el-form :model="editRoleForm" ref="editRoleFormRef" label-width="70px">
        <el-form-item label="角色名称">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleEialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 添加角色 -->
    <el-dialog
     title="添加角色"
      :visible.sync="addRoleDialogVisible"
      width="50%"
      @close="addRoleDialogClosed">
      <!-- 主体内容区域 -->
      <el-form
       :model="addRoleForm"
       ref="addRoleRef"
       label-width="70px">
       <el-form-item label="角色名称" prop="roleName">
         <el-input v-model="addRoleForm.roleName"></el-input>
       </el-form-item>
       <el-form-item label="角色描述" prop="roleDesc">
         <el-input v-model="addRoleForm.roleDesc"></el-input>
       </el-form-item>
      </el-form>
       <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>

    </el-dialog>

    <!-- 分配权限的对话框 -->
    <el-dialog
  title="分配权限"
  :visible.sync="setRightDialogVisible"
  width="50%" @close="setRightDialogClosed">
  <!-- 树形控件 -->
 <el-tree show-checkbox :data="rightslist" :props="treeProps" node-key="id" default-expand-all
 :default-checked-keys="defKeys" ref="treeRef"></el-tree>
  <span slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
</el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //所有角色列表数据
      rolelist: [],
      editRoleEialogVisible: false,
      editRoleForm: {},
        //   控制添加角色对话框的显示与隐藏
      addRoleDialogVisible: false,
      addRoleForm:{
        roleName:'',
        roleDesc:''
      },
      //控制分配权限对话框的显示与隐藏
      setRightDialogVisible:false,
      //所有权限的数据
      rightslist:[],
      //树形控件的绑定对象
      treeProps:{
        label:'authName',
        children:'children'
      },
      // 默认选中的节点Id值数组
      defKeys:[],
      //当前即将分配权限的角色id
      roleId:''
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    // //所有角色列表数据
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表数据失败')
      }
      this.rolelist = res.data
      console.log(this.rolelist)
    },

    //展示编辑角色的对话框
    async showEditDialog(id) {
      console.log(id)
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询角色信息失败')
      }

      this.editRoleForm = res.data
      this.editRoleEialogVisible = true
    },

    //修改角色信息，点击“确定”按钮，提交信息
    async editRoleInfo() {
      //发起修改角色信息的数据请求 put请求
      const { data: res } = await this.$http.put(
        'roles/' + this.editRoleForm.roleId,
        {
          roleName: this.editRoleForm.roleName,
          roleDesc: this.editRoleForm.roleDesc,
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('更新角色信息失败')
      }
      //关闭对话框
      this.editRoleEialogVisible = false
      //刷新角色列表
      this.getRolesList()
      //提示修改成功
      this.$message.success('更新角色信息成功')
    },
    //监听修改用户对话框的关闭事件
    editDialogClosed() {
      //拿到表单的引用
      this.$refs.editRoleFormRef.resetFields()
    },

     //根据id删除对应的角色信息
    async removeRoleById(id){
        console.log(id)
        //弹框询问是否删除
        const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err =>{
            return err
        })
        ////promise 返回值是promise 可以用async await优化
        //如果用户确认删除，则返回值为字符串 confirm
        //如果用户取消删除，则返回值为字符串 cancel
        console.log(confirmResult) 
        if(confirmResult !== 'confirm') {
            return this.$message.info('已取消删除')
        }
        //删除用户  delete请求
        const {data:res} = await this.$http.delete('roles/' + id)
        if(res.meta.status !== 200){
            return this.$message.error("删除用户失败")
        }
        this.$message.success('删除用户成功')
         this.getRolesList()
    },
     //监听添加角色对话框的关闭事件
    addRoleDialogClosed() {
      //拿到表单的引用
      this.$refs.addRoleRef.resetFields()
    },
     //点击“确定”按钮，添加新角色
   async addRole(){
     const {data:res} = await this.$http.post('roles',this.addRoleForm)
     if(res.meta.status !== 201){
       this.$message.error("添加角色失败！")
     }
     this.$message.success("添加角色成功！")
     this.addRoleDialogVisible = false
     //重新获取角色列表
     this.getRolesList()
    },
    //根据id删除对应的权限
    async removeRightById(role,rightId){
      //弹框提示用户是否要删除
       const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)

        if(confirmResult !== 'confirm'){
          return this.$message.info('取消了删除！')
        }

      const {data:res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if(res.meta.status !== 200){
        return this.$message.error('删除权限失败')
      }
      // this.getRolesList()
      //防止点击确定，删除后整个刷新
      role.children = res.data
    },
  
      //展示分配权限的对话框
      async showSetRightDialog(role){
        this.roleId = role.id
        //获取所有权限的数据
        const {data:res} = await this.$http.get('rights/tree')
        
        if(res.meta.status != 200){
          return this.$message.error('获取数据失败！')
        }
        //把获取到的权限数据保存到data中
        this.rightslist = res.data
        console.log(this.rightslist)

        //递归获取三级节点的Id
        this.getLeafKeys(role,this.defKeys)

        this.setRightDialogVisible = true

      },

      //通过递归的形式，获取角色下所有的三级权限的id，并保存到defKeys数组中
      getLeafKeys(node,arr){
        //如果当前node节点不包含children属性，则是三级节点
        if(!node.children){
          return arr.push(node.id)
        }

        node.children.forEach(item =>
          this.getLeafKeys(item,arr))
      },
      
      //监听分配权限对话框的关闭事件
      setRightDialogClosed(){
        this.defKeys = []
      },

      //点击为角色分配权限
      async allotRights(){
        const Keys = [
          //获取已选择节点的id数组
          ...this.$refs.treeRef.getCheckedKeys(),
          ...this.$refs.treeRef.getHalfCheckedKeys()
        ]

        console.log(Keys)
        const idStr = Keys.join(',')

      const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})

      if(res.meta.status != 200){
        return this.$message,error('分配权限失败')
      }

      this.$message.success('分配权限成功')

      this.getRolesList()
      this.setRightDialogVisible = false

      }
  }
}
</script>

<style lang="less" scoped>
.el-tag{
  margin: 7px;
}

.bdtop{
  border-top: 1px solid #eee;
}
.bdbottom{
   border-bottom: 1px solid #eee;
}

.vcenter{
  display:flex;
  align-items: center;
}
</style>