<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
          <el-breadcrumb-item>权限管理</el-breadcrumb-item>
          <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>

         <!-- 卡片视图区域  -->
         <el-card>
            <!-- 添加角色按钮区域 -->
            <el-row>
                <el-col>
                    <el-button type="primary" @click="addRolesVisible = true">添加角色</el-button>
                </el-col>
            </el-row>

            <!-- 角色列表区域 -->
            <el-table :data="roleList" border stripe>
                <!-- 展开列 -->
                <el-table-column type="expand">
                    <template slot-scope="scope">
                        <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
                            <el-col :span="5">
                                <!-- 渲染一级权限 -->
                                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 渲染二级和三级权限 -->
                            <el-col :span="19">
                               <!-- 通过for循环嵌套渲染二级权限 -->
                                <el-row v-for="(item2, i2) in item1.children" :key="item2.id" :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']">
                                    <el-col :span="6">
                                        <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                                         <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span="18">
                                        <el-tag type="warning" v-for="(item3, i3) in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">
                                            {{item3.authName}}
                                        </el-tag>
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
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRolesById(scope.row.id)">删除</el-button>
                        <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>
         </el-card>


            <!-- 添加角色 -->
            <el-dialog
                    title="添加角色"
                    :visible.sync="addRolesVisible"
                    width="50%"
                    @close="addRolesClosed"
                    >
                <el-form ref="addRolesRef" :model="addRolesForm" label-width="80px" :rules="addRolesRules">
                    <el-form-item label="角色名称" prop="roleName">
                      <el-input v-model="addRolesForm.roleName" ></el-input>
                    </el-form-item>
                    <el-form-item label="角色描述" prop="roleDesc">
                      <el-input v-model="addRolesForm.roleDesc"></el-input>
                    </el-form-item>
                </el-form>
                <span slot="footer" class="dialog-footer">
                  <el-button @click="addRolesVisible = false">取 消</el-button>
                  <el-button type="primary" @click="addRoleInfo">确 定</el-button>
                </span>
            </el-dialog>

            <!-- 编辑角色 -->
            <el-dialog
                    title="编辑角色"
                    :visible.sync="editRolesVisible"
                    width="50%"
                    @close="editRolesClosed"
                    >
                <el-form ref="editRolesRef" :model="editRolesForm" label-width="80px" :rules="editRolesRules">
                    <el-form-item label="角色名称" prop="roleName">
                      <el-input v-model="editRolesForm.roleName" ></el-input>
                    </el-form-item>
                    <el-form-item label="角色描述" prop="roleDesc">
                      <el-input v-model="editRolesForm.roleDesc"></el-input>
                    </el-form-item>
                </el-form>
                <span slot="footer" class="dialog-footer">
                  <el-button @click="editRolesVisible = false">取 消</el-button>
                  <el-button type="primary" @click="editRoleInfo">确 定</el-button>
                </span>
            </el-dialog>

            <!-- 分配权限的对话框 -->
            <el-dialog
                title="分配权限"
                :visible.sync="setRightDialogVisible"
                width="50%"
                @close="setRightDialogClosed"
                >
                <!-- 树形控件 -->
                <!-- node-key 指定选中的是id值
                    default-expand-all 默认展开所有节点
                    default-checked-keys 默认勾选的节点 -->
                <el-tree :data="rightsList" :props="treeProps" show-checkbox
                node-key="id" default-expand-all
                :default-checked-keys="defKeys"
                ref="treeRef"></el-tree>
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
            // 所有角色列表数据
            roleList: [],
            addRolesVisible: false,
            addRolesForm: {
                roleName: '',
                roleDesc: ''
            },
            addRolesRules: {
                roleName: [
                 { required: true, message: '请输入角色名称', trigger: 'blur' },
                 { min: 3, max: 10, message: '角色名称的长度在3 ~ 10 之间', trigger: 'blur' }
                ],
                roleDesc:[
                    { required: true, message: '请输入角色描述', trigger: 'blur' },
                    { min: 5, max: 20, message: '角色描述的长度在5 ~ 20 之间', trigger: 'blur' }
                ]
            },
            editRolesVisible: false,
            // 存储根据id查询出来的角色数据对象
            editRolesForm: {},
            editRolesRules: {
                roleName: [
                 { required: true, message: '请输入角色名称', trigger: 'blur' },
                 { min: 3, max: 10, message: '角色名称的长度在3 ~ 10 之间', trigger: 'blur' }
                ],
                roleDesc:[
                    { required: true, message: '请输入角色描述', trigger: 'blur' },
                    { min: 5, max: 20, message: '角色描述的长度在5 ~ 20 之间', trigger: 'blur' }
                ]
            },
            // 控制分配权限对话框的显示与隐藏
            setRightDialogVisible: false,
            // 所有权限的数据
            rightsList: [],
            // 树形控件的属性绑定对象
            treeProps: {
                label: 'authName',
                children: 'children'
            },
            // 默认选中的节点Id值
            defKeys: [],
            // 当前即将分配权限的id
            roleId: ''
        }
    },
    created() {
        this.getRolesList();
    },
    methods: {
        async getRolesList() {
            const {data: res} = await this.$http.get('roles')
            if(res.meta.status !== 200) this.$message.error('获取角色列表失败！')
            this.roleList = res.data
        },
        // 监听角色弹窗关闭
        addRolesClosed() {
            this.$refs.addRolesRef.resetFields()
        },
        // 添加角色
        addRoleInfo() {
            this.$refs.addRolesRef.validate(async valid => {
                if(!valid) return;
                const {data: res} = await this.$http.post('roles', this.addRolesForm)
                if(res.meta.status !== 201) return this.$message.error('添加用户角色失败')
                this.$message.success('添加角色成功！')
                this.addRolesVisible = false
                this.getRolesList()  
            })
        },
        async showEditDialog(id) {
            const {data: res} = await this.$http.get(`roles/${id}`)
            if(res.meta.status !== 200) return this.$message.error('获取角色信息失败')
            this.editRolesForm = res.data
            this.editRolesVisible = true
            
        },
        editRolesClosed() {
            this.$refs.editRolesRef.resetFields()
        },
        editRoleInfo() {
            this.$refs.editRolesRef.validate(async valid => {
                if(!valid) return
                const {data: res} = await this.$http.put(`roles/${this.editRolesForm.roleId}`,{
                    roleName: this.editRolesForm.roleName,
                    roleDesc: this.editRolesForm.roleDesc
                })
                if(res.meta.status !== 200) return this.$message.error('修改角色信息失败')
                this.$message.success('修改角色成功！')
                this.editRolesVisible = false
                this.getRolesList()
            })
        },
        async removeRolesById(id) {
            const confirmResult = await this.$confirm('此操作将永久删除该角色，是否继续？','提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            if(confirmResult !== 'confirm') return this.$message.info('已经取消删除')
            const {data: res} = await this.$http.delete(`roles/${id}`)
            if(res.meta.status !== 200) this.$message.error('删除角色信息失败')
            this.$message.success('删除角色成功！')
            this.getRolesList()
        },
        // 根据id删除对应的权限
        async removeRightById(role, rightId) {
            // 弹框提示是否删除
         const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
           confirmButtonText: '确定',
           cancelButtonText: '取消',
           type: 'warning'
         }).catch(err => err)
         if(confirmResult !== 'confirm') return this.$message.info('取消了删除')
         const {data: res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
         if(res.meta.status !== 200) this.$message.error('删除权限失败')

         // 防止删除后 列表刷新导致 折叠卡合并  
         // role 为传过来的 那一行数据 res.data返回的是删除后的最新数据
         role.children = res.data
        },
        // 展示分配权限的对话框
        async showSetRightDialog(role) {
            // 保存id 给保存树的时候  需要传id
            this.roleId = role.id
            // 获取所有权限的数据
            const {data: res} = await this.$http.get('rights/tree')
            if(res.meta.status !== 200) this.$message.error('获取权限数据失败')
            //把获取到的权限数据保存到 data中
            this.rightsList = res.data

            // 递归获取三级节点的id
            this.getLeafKeys(role, this.defKeys)
            this.setRightDialogVisible = true

            
        },
        // 通过递归的形式，获取角色下所有三级权限的id，并保存到defKeys
         // 点击分配权限按钮同时，将当前角色身上所有三级权限的id获取出来放到defKeys
        getLeafKeys(node, arr) {
           // 如果当前 node 节点不包含children属性，则是三级节点
            if(!node.children) {
                return arr.push(node.id)
            }

            // 不是三级节点，循环当前node里面所有数组
            node.children.forEach(item => {
                this.getLeafKeys(item, arr)
            })

        },
        // 监听分配权限的对话框的关闭事件
        // 解决多次打开树 数据累加问题
        setRightDialogClosed() {
            this.defKeys = []
        },
        // 点击为角色分配权限
        async allotRights() {
            const keys = [
                // 获取所有已选中的节点
                ...this.$refs.treeRef.getCheckedKeys(),
                // 获取选中的父级半节点
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ]
            const idStr = keys.join(',')
            const {data: res} = await this.$http.post(`roles/${this.roleId}/rights`, {
                rids: idStr
            })
            if(res.meta.status !== 200) this.$message.error('分配权限失败')
            this.$message.success('分配权限成功！')
            this.getRolesList()
            this.setRightDialogVisible = false
        }
    }
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