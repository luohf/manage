<template>
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <el-row>
        <!-- 添加角色按钮区域  -->
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
<el-table :data="RolesList" border stripe>
    <!-- 展开列 -->
    <el-table-column type="expand" >
        <template slot-scope="scope">
            <el-row :class="['bdbottom',i1 === 0 ? 'bdtop':'','vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable
              @close = "removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
                </el-col>
                <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <!-- 渲染二级权限 -->
                <el-row :class="[i3 === 0? '':'bdtop','vcenter']" v-for="(item3,i3) in item1.children" :key="item3.id"
                >
                  
          <el-col :span="6">
            <el-tag type="success" closable
              @close = "removeRightById(scope.row,item3.id)"> {{item3.authName}}</el-tag>
            <i class="el-icon-caret-right"></i>
          </el-col>
          <el-col :span="18">
              <el-tag type="warning" v-for="(item6,i6) in item3.children" :key="item6.id" closable
              @close = "removeRightById(scope.row,item6.id)">
                {{item6.authName}}>
                </el-tag>
          </el-col>
                </el-row>
              </el-col>
            </el-row>
            <!-- <pre>{{scope.row}}</pre> -->
            
        </template>
    </el-table-column>
    <!-- 索引列 -->
<el-table-column type="index"></el-table-column>
<el-table-column label="角色" prop="roleName"></el-table-column>
<el-table-column label="角色描述" prop="roleDesc"></el-table-column>
<el-table-column label="操作" width="300px">
   <template slot-scope="scope">
        <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
        <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
        <!-- <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button> -->
    </template>

</el-table-column>
</el-table>
    </el-card>

<!-- 分配权限的弹出框 -->
<el-dialog
  title="提示"
  :visible.sync="SetRightDialogVisible"
  width="50%" @close="setRightDialogClosed"
 >

 <!-- 树形控件 -->
  <el-tree :data="rightslist"  show-checkbox :props="treeProps" default-expand-all node-key="id" :default-checked-keys="defKeys" ref="treeRef"></el-tree>
  <span slot="footer" 
  class="dialog-footer">
    <el-button @click="showSetRightDialog">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
</el-dialog>
  </div>
</template>


<script>
export default {
  data() {
    return {
      RolesList: [],
      // 控制分配权限弹出框的出现与隐藏
      SetRightDialogVisible:false,
      // 权限
      rightslist:[],
      // 树形结构
      treeProps:{
        children:'children',
        label:'authName'

      },
      // 选中的id节点数组
      defKeys:[],

      // 保存角色id
      roleId:''
    };
  },
  created() {
    this.getRolesList();
  },
  methods: {
    // 获取角色列表
    async getRolesList() {
      const { data: res } = await this.$http.get("roles");
      if (res.meta.status !== 200) {
        return this.$message.error("获取角色列表失败");
      }
      this.RolesList = res.data;
      console.log(this.RolesList);
    },
    async removeRightById(role,rightId){
      const result = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch((error) => {
          return error
        })
        console.log(result)
        if(result === 'cancel'){
          this.$message.info('取消了删除')
        }
          const {data:res} = await this.$http.delete
          (`roles/${role.id}/rights/${rightId}`)
          if(res.meta.status !== 200){
            return this.$message.error("删除权限失败")

          }else{
            // 难点
            // 返回数据，不用再去调用列表刷新页面
            role.children = res.data
          }
       
    },
    // 分配权限
    async showSetRightDialog(role){
      this.roleId = role.id
      console.log(this.roleId)
      this.SetRightDialogVisible = true
      const {data:res} = await this.$http.get('rights/tree')
      if(res.meta.status !== 200){
        return this.$message.error('分配权限失败')
      }
      // 获取到的权限数据保存到data
      this.rightslist = res.data
      console.log(this.rightslist)
      // 递归调用三级节点的id
      console.log(role)
      // 递归获取三级节点的id
      this.getLeafKeys(role,this.defKeys)
      this.SetRightDialogVisible = true
      
    },
    // 用递归的形式，获取角色下所有三级权限的id,并保存到defKeys数组
   
    getLeafKeys(node,arr) {
       // 如果当前node节点没有children属性，则是三级节点
      if(!node.children) {
        return arr.push(node.id)
      }

// 因为传过来的node是一个对象，但node.children是一个数组，这边需要数组
      node.children.forEach(item => {
        this.getLeafKeys(item,arr)
      });
      
    },
    // 关闭弹出框
    setRightDialogClosed() {
      // 清空id
      this.defKeys=[]
    },
    // 分配权限
    async allotRights() {

      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      console.log(keys)

      const idStr = keys.join(',')

      const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})
      if(res.meta.status !== 200)
      return this.$message.error('分配权限失败')
      this.getRolesList()
      this.SetRightDialogVisible = false
    }
  }
};
</script>
<style lang="less" scoped>
.el-tag{
  margin:7px;
}
.bdtop {
  border-top:1px solid #eee;
}

.bdbottom {
  border-bottom:1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>