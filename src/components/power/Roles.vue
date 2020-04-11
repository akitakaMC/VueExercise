<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>
        <a href="/">权限管理</a>
      </el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
        <!-- 添加角色按钮区 -->
        <el-row>
            <el-col>
                <el-button type="primary" @click="addRole">添加角色</el-button>
            </el-col>
        </el-row>
      <el-table :data="rolesList" border stripe>
        <el-table-column type="expand">
            <template slot-scope="scope">
                <el-row :class="['bdbuttom', i1===0?'bdtop':'','vcenter']" :key="item1.id" v-for="(item1,i1) in scope.row.children">
                    <!-- 一级权限 -->
                    <el-col :span="5">
                        <el-tag closable @close="removeRightById(scope.row,item3.id)">{{ item1.authName}}</el-tag>
                        <i class="el-icon-caret-right"></i>
                    </el-col>
                    <!-- 其他权限 -->
                    <el-col :span="19">
                        <!-- 渲染所有二级权限 -->
                        <el-row :class="[i2===0?'':'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                            <el-col :span="6">
                                <el-tag type="success" closable @close="removeRightById(scope.row,item3.id)">{{item2.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <el-col :span="18">
                                <el-tag v-for="item3 in item2.children" :key="item3.id" type="warning" closable @close="removeRightById(scope.row,item3.id)">{{item3.authName}}</el-tag>
                            </el-col>
                        </el-row>
                    </el-col>
                </el-row>
            </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
            <template slot-scope="scope">
                <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDialog(scope.row.id)">编辑</el-button>
                <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeRoleById(scope.row.id)">删除</el-button>
                <el-button type="warning" size="mini" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
            </template>
        </el-table-column>
      </el-table>
    </el-card>
    <el-dialog title="修改角色信息" :visible.sync="editDialogVisible"
    width="50%" @close="editDialogClosed">
    <el-form :model="editForm" :rules="editFormRules"
    ref="editFormRef" label-width="90px">
    <el-form-item label="角色名称" prop="roleName">
      <el-input v-model="editForm.roleName"></el-input>
    </el-form-item>
    <el-form-item label="角色描述" prop="roleDesc">
      <el-input v-model="editForm.roleDesc"></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="editDialogVisible=false">取 消</el-button>
      <el-button type="primary" @click="editRoleInfo">确 定</el-button>
    </span>
    </el-dialog>
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible"
    width="50%" @close="setRightDialogClosed">
    <!-- 树形控件 -->
    <el-tree :data="rightsList" :props="treeProps" show-checkbox
    node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
    <span slot="footer" class="dialog-footer">
      <el-button @click="setRightDialogVisible=false">取 消</el-button>
      <el-button type="primary" @click="allotRights">确 定</el-button>
    </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: [],
      editDialogVisible: false,
      editForm: [],
      editFormRules: {
        roleName: [{ required: true, message: '请输入角色名称', trigger: 'blur' }, { min: 2, max: 10, message: '角色名称的长度在2-10个字符之间', trigger: 'blur' }],
        roleDesc: [{ required: true, message: '请输入角色描述', trigger: 'blur' }, { min: 3, max: 10, message: '角色描述长度应在3-15位直接', trigger: 'blur' }]
      },
      setRightDialogVisible: false,
      rightsList: [],
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      defKeys: [],
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
      this.rolesList = res.data
    },
    addRole () {

    },
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败！')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    editRoleInfo () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        // 发起修改用户请求
        const { data: res } = await this.$http.put('roles/' + this.editForm.roleId, { roleName: this.editForm.roleName, roleDesc: this.editForm.roleDesc })
        if (res.meta.status !== 200) {
          return this.$message.error('更新用户信息失败')
        }
        this.editDialogVisible = false
        this.getRolesList()
        this.$message.success('更新用户信息成功')
      })
    },
    async removeRoleById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success('删除成功')
    },
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm('是否删除该项权限', '提示', {
        confirmButtomText: '确定',
        cancelButtomText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success('删除失败')
      role.children = res.data
    },
    async showSetRightDialog (role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败！')
      }
      this.rightsList = res.data
      // 递归获取三级节点的id
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归获取所有选中的三级节点的id
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item =>
        this.getLeafKeys(item, arr))
    },
    setRightDialogClosed () {
      this.defKeys = []
    },
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('修改权限失败')
      }
      this.setRightDialogVisible = false
      this.getRolesList()
      this.$message.success('修改权限成功')
      // console.log(res)
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag{
    margin: 7px;
    }
.bdtop{
    border-top: 1px solid #eeeeee;
}
.bdbuttom{
    border-bottom: 1px solid #eeeeee;
}
.vcenter{
    display: flex;
    align-items: center;
}
</style>
