<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true"
            >添加角色</el-button
          >
        </el-col>
      </el-row>
      <!-- 角色列表区 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
            >
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightById(scope.row, item1.id)"
                  >{{ item1.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二级和三级权限 -->
              <el-col :span="19">
                <el-row
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                >
                  <!-- 二级权限 -->
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级权限 -->
                  <el-col :span="18">
                    <el-tag
                      v-for="(item3, i3) in item2.children"
                      :key="item3.id"
                      type="warning"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeRoleById(scope.row.id)"
              >删除</el-button
            >
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showSetRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>

      <!-- 编辑角色的对话框 -->
      <el-dialog
        title="编辑角色"
        :visible.sync="EditDialogVisible"
        width="50%"
        @close="EditDialogClosed"
      >
        <el-form
          :model="editForm"
          :rules="editFormRules"
          ref="editFormRef"
          label-width="80px"
        >
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="editForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述">
            <el-input v-model="editForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>

        <span slot="footer" class="dialog-footer">
          <el-button @click="EditDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editRoleInfo">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 添加角色的对话框 -->
      <el-dialog
        title="添加角色"
        :visible.sync="addDialogVisible"
        width="50%"
        @close="addDialogClosed"
      >
        <!-- 内容主体区 -->
        <el-form
          :model="addForm"
          :rules="addFormRules"
          ref="addFormRef"
          label-width="80px"
        >
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="addForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="addForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false"> 取 消 </el-button>
          <el-button type="primary" @click="addRole">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>

    <!-- 分配权限的对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed"
    >
      <!-- 树形控件 -->
      <!-- node-key="id" 意思是把 树中节点的key设置成了角色id -->
      <el-tree
        :data="rightslist"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>

      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Roles',
  data() {
    return {
      //  所有角色列表数据
      rolelist: [],
      //   控制添加角色对话框
      addDialogVisible: false,
      //   控制编辑角色的 对话框
      EditDialogVisible: false,
      //   添加的角色信息
      addForm: {
        roleName: '',
        roleDesc: '',
      },
      //  查询到的角色信息
      editForm: {},
      //   添加角色信息的表单验证规则
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '用户名长度在3 到 10 个字符',
            trigger: 'blur',
          },
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
        ],
      },
      // 修改的表单验证规则
      editFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '用户名长度在3 到 10 个字符',
            trigger: 'blur',
          },
        ],
      },
      // 控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      // 所有权限的数据
      rightslist: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children',
      },
      // 默认选中的节点Id值数组
      defKeys: [],
      // 当前即将分配角色的id
      roleId: '',
    }
  },
  created() {
    // 获取角色列表
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      //   发送请求
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.erroe('获取用户列表失败')
      }
      // 保存获取的数据
      this.rolelist = res.data
    },
    // 展示编辑角色的对话框
    async showEditDialog(id) {
      // 发送请求
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询角色信息失败！')
      }
      // 将查询到的信息保存
      this.editForm = res.data
      // 展示编辑角色的对话框
      this.EditDialogVisible = true
    },
    // 监听编辑角色对话框关闭事件
    EditDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 编辑角色信息的提交
    editRoleInfo() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          'roles/' + this.editForm.roleId,
          {
            roleName: this.editForm.roleName,
            roleDesc: this.editForm.roleDesc,
          }
        )
        console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('编辑角色信息失败！')
        }

        this.EditDialogVisible = false
        this.getRolesList()
        this.$message.success('角色信息编辑成功！')
      })
    },
    // 根据id删除指定的角色
    async removeRoleById(id) {
      // 询问用户是否确认删除操作
      const confirmResult = await this.$confirm(
        '此操作将永久删除该角色，是否继续？',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }
      ).catch((err) => err)
      // 如果用户确认删除 则会返回confirm
      // 取消则返回 cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已经取消删除操作！')
      }
      // 表示用户确定 发送删除的请求
      //    发送删除的请求
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败！')
      }
      this.$message.success('删除用户成功！')
      this.getRolesList()
    },
    // 监听添加角色对话框关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 添加新角色
    addRole() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        // 发送添加角色的网路请求
        const { data: res } = await this.$http.post('roles', this.addForm)
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败！')
        }
        this.$message.success('添加角色成功')
        this.addDialogVisible = false
        this.getRolesList()
      })
    },
    // 根据id删除对应权限
    async removeRightById(role, rightId) {
      // 提示用户是否需要删除
      const confirmResult = await this.$confirm(
        '此操作将永久删除该权限，是否继续？',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }
      ).catch((err) => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除！')
      }

      // 注：此接口返回的data是该角色删除权限后 最新的权限列表
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )

      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败！')
      }

      this.$message.success('删除权限成功！')
      // 为了防止每次删除权限后，页面刷新，展开栏自动合上 不调用 this.getRolesList()
      // 而是直接将服务器返回的最新权限 赋值给 role.children
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')

      if (res.meta.status !== 200) {
        return this.$message.error('获取角色权限失败！')
      }
      // 保存获取到的权限数据
      this.rightslist = res.data
      // 保存获取到的角色id 为了后面使用
      this.roleId = role.id 
      // 递归获取三级权限的id
      this.getLeafKeys(role,this.defKeys)

      this.setRightDialogVisible = true
    },
    //通过递归形式获取角色下所有三级权限并保存
    getLeafKeys(node,arr) {
      // 如果当前node是三级节点
      if(!node.children){
        return arr.push(node.id)
      }

      // 递归
      node.children.forEach(item => {
        this.getLeafKeys(item,arr)
      })
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed() {
      // 关闭时清空defkeys
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights() {
      
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      
      const idStr = keys.join(',')
      // 发送分配权限的请求
      const {data: res } = await this.$http.post(`roles/${this.roleId}/rights`,{
        rids: idStr
      })

      if(res.meta.status !== 200 ){
        return this.$message.error('分配权限失败！')
      }
      this.$message.success('分配权限成功！')

      // 刷新角色列表
      this.getRolesList()
      // 关闭对话框
      this.setRightDialogVisible = false
    }
  },
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>

