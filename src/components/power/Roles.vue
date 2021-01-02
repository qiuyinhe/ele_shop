<template>
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 添加按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRoleDia = true"
            >
            添加角色
            </el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
        <el-table :data="rolesList" border stripe>
        <!-- 这是展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope" >
            <el-row  :class="['bdbottom',i1 == 0 ? 'bdtop' : '','vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
              <el-col :span="5">
                <el-tag closable>
                {{item1.authName}}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row   :class="[i2 == 0 ? '' : 'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                   <el-col :span="6">
                      <el-tag closable type="success">{{item2.authName}}</el-tag>
                      <i class="el-icon-caret-right"></i>
                   </el-col>
                   <el-col :span="18">
                     <el-tag closable @close="removeRightById(scope.row,item3.id)" type="warning" v-for="(item3,i3) in item2.children" :key="item3.id">{{item3.authName}}</el-tag>
                   </el-col>
                </el-row>
              </el-col>
            </el-row>
            <!-- <pre>
            {{scope.row}}
            </pre> -->
          </template>
        </el-table-column>
        <!-- 这是索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">  
          <template slot-scope="scope">
            <el-button
              icon="el-icon-edit"
              type="primary"
              size="mini"
              @click="updataRole(scope.row.id)"
            >
            编辑
            </el-button>
            <el-button type="warning" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
            <el-button type="danger" size="mini" @click="deleRoleById(scope.row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加用户的dia -->
    <el-dialog title="添加用户" :visible.sync="addRoleDia" width="30%">
      <el-form ref="form" :model="newRole" label-width="80px">
        <el-form-item label="角色名称">
          <el-input v-model="newRole.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="newRole.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDia = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑用户的dia -->
    <el-dialog title="更改用户信息" :visible.sync="seleRoleDia" width="30%">
        <el-form ref="selRoleRef" :model="selRole" label-width="80px">
          <el-form-item label="角色 ID">
            <el-input v-model="selRole.roleId" disabled></el-input>
          </el-form-item>
          <el-form-item label="角色名称">
            <el-input v-model="selRole.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述">
            <el-input v-model="selRole.roleDesc"></el-input>
          </el-form-item>
        </el-form>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="seleRoleDia = false">取 消</el-button>
        <el-button type="primary" @click="cfmEditRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限的对话框 -->
    <el-dialog
        title="分配权限"
        :visible.sync="setRightDialogVisible"
        width="30%"
       @close="setRightDialogClose">
      <el-tree :data="rightList" :props="rightTreeProps" 
          show-checkbox 
          node-key='id' 
          default-expand-all 
          :default-checked-keys='defKeys'
          ref="rightTreeRef">
      </el-tree>
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
      rolesList: [],
      addRoleDia: false,
      seleRoleDia: false,
      newRole: {
        roleName: "老夫子",
        roleDesc: "新来的老板",
      },
      selRole: {},
      setRightDialogVisible: false,
      rightList: [],
      rightTreeProps: {
        children: "children",
        label: "authName",
      },
      defKeys: [],
      roleId: "",
    };
  },
  created() {
    this.getRolesList();
  },
  methods: {
    //获取角色信息
    async getRolesList() {
      const { data: res } = await this.$http.get("roles");
      // console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg);
      }
      this.rolesList = res.data;
      console.log(this.rolesList);
    },
    // 添加角色
    async addRole() {
      // console.log(11);
      const { data: res } = await this.$http.post("roles", this.newRole);
      if (res.meta.status !== 201) {
        return this.$message(res.meta.msg);
      }
      this.$message(res.meta.msg);
      this.addRoleDia = false;
      this.getRolesList();
    },
    // 查询编辑角色信息
    async updataRole(id) {
      // this.getRolesList();
      this.seleRoleDia = true;
      // console.log(id);
      const { data: res } = await this.$http.get("roles/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg);
      }
      this.selRole = res.data;
    },
    //确定修改用户信息
    async cfmEditRole() {
      this.seleRoleDia = false;
      const { data: res } = await this.$http.put(
        `roles/${this.selRole.roleId}`,
        {
          roleName: this.selRole.roleName,
          roleDesc: this.selRole.roleDesc,
        }
      );
      if (res.meta.status !== 200) {
        this.$message.error(res.meta.msg);
      }
      // console.log(res);
      this.getRolesList();
    },
    // 根据id删除角色
    deleRoleById(id) {
      this.$confirm("此操作将永久删除该角色, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http.delete("roles/" + id);
          this.getRolesList();
          this.$message({
            type: "success",
            message: "删除成功!",
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    // 根据id删除对应的权限
    async removeRightById(role, rightId) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该文件, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      );
      if (confirmResult !== "confirm") {
        return this.$message.info("用户取消了删除");
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      );
      // console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg);
      }
      this.$message.success(res.meta.msg);
      role.children = res.data;
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id;
      //获取权限的数据
      const { data: res } = await this.$http.get("rights/tree");
      if (res.meta.status != 200) {
        return this.$message.error(res.meta.msg);
      }
      this.rightList = res.data;
      // console.log(role);
      this.getLeafKeys(role, this.defKeys);
      this.setRightDialogVisible = true;
    },
    //通过递归的形式获取所有权限下的三级权限的id，并保存到defKeys数组中
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id);
      }
      node.children.forEach((item) => {
        this.getLeafKeys(item, arr);
      });
    },
    setRightDialogClose() {
      this.defKeys = [];
    },
    //点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.rightTreeRef.getCheckedKeys(),
        ...this.$refs.rightTreeRef.getHalfCheckedNodes(),
      ];
      // console.log(keys);
      const idStr = keys.join(",");
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        {
          rids: idStr,
        }
      );
      this.getRolesList();
      this.setRightDialogVisible = false;
    },
  },
};
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