<!--
 * @Author: your name
 * @Date: 2020-10-31 08:40:22
 * @LastEditTime: 2020-11-01 20:28:06
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \vue介绍\mallmanage52\src\components\users\users.vue
-->
<template>
<el-card class="box-card">
  <!--1.面包屑-->
  <!--/首页/用户管理/用户列表-->
  <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item>首页</el-breadcrumb-item>
    <el-breadcrumb-item>用户管理</el-breadcrumb-item>
    <el-breadcrumb-item>用户列表</el-breadcrumb-item>
  </el-breadcrumb>
  <!--2.搜索-->
  <el-row class="searchRow">
    <el-col>
      <el-input clearable placeholder="请输入内容" v-model="query" class="inputSearch" @clear="loadUserList()">
        <el-button @click="searchUser()" slot="append" icon="el-icon-search"></el-button>
      </el-input>
      <el-button type="success" @click="showAddUserDia()">添加用户</el-button>
    </el-col>
  </el-row>

  <!--3.表格-->
  <el-table :data="userList" style="width: 100%">
    <el-table-column label="#" width="60" type="index"></el-table-column>
    <el-table-column prop="username" label="姓名" width="80"></el-table-column>
    <el-table-column prop="email" label="邮箱"></el-table-column>
    <el-table-column prop="mobile" label="电话"></el-table-column>
    <el-table-column label="创建时间">
      <!-- 如果单元格内显示的内容不是字符串（文本），需要给被显示的内容外层包裹一个template-->
      <!--template内部使用数据设置slot-scope属性，该属性的值要用数据create_time的数据源userlist-->
      <!-- slot-scope的值userlist其实就是el-table绑定的数据userlist,userlist.row->数组的每一个对象-->
      <template slot-scope="userlist">{{userlist.row.create_time | fmtdate}}</template>
    </el-table-column>
    <el-table-column label="用户状态">
      <!--用户状态-->
      <template slot-scope="scope">
        <el-switch @change="changeMgState(scope.row)" v-model="scope.row.mg_state" active-color="#13ce66" inactive-color="#ff4949"></el-switch>
      </template>
    </el-table-column>
    <el-table-column prop="openid" label="操作">
      <template slot-scope="scope">
        <el-button size="mini" plain type="primary" icon="el-icon-edit" circle @click="showEditUserDia(scope.row)"></el-button>
        <el-button size="mini" plain type="success" icon="el-icon-check" circle @click="showSetUserRoleDia(scope.row)"></el-button>
        <el-button size="mini" plain type="danger" icon="el-icon-delete" circle @click="showDeleUserMsgBox(scope.row.id)"></el-button>
      </template>
    </el-table-column>
  </el-table>
  <!--4.分页-->
  <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="pagenum" :page-sizes="[2,4,6,8]" :page-size="pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total"></el-pagination>
  <!--5对话框-->
  <!--添加用户对话框-->
  <el-dialog title="添加用户" :visible.sync="dialogFormVisibleAdd">
    <el-form :model="form">
      <el-form-item label="用户名" label-width="100px">
        <el-input v-model="form.username" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="密码" label-width="100px">
        <el-input v-model="form.password" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="邮箱" label-width="100px">
        <el-input v-model="form.email" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="电话" label-width="100px">
        <el-input v-model="form.mobile" autocomplete="off"></el-input>
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="dialogFormVisibleAdd = false">取 消</el-button>
      <el-button type="primary" @click="addUser()">确 定</el-button>
    </div>
  </el-dialog>
  <!--编辑用户对话框-->
  <el-dialog title="编辑用户" :visible.sync="dialogFormVisibleEdit">
    <el-form :model="form">
      <el-form-item label="用户名" label-width="100px">
        <el-input v-model="form.username" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="邮箱" label-width="100px">
        <el-input v-model="form.email" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="电话" label-width="100px">
        <el-input v-model="form.mobile" autocomplete="off"></el-input>
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="dialogFormVisibleEdit = false">取 消</el-button>
      <el-button type="primary" @click="editUser()">确 定</el-button>
    </div>
  </el-dialog>
  <!--分配角色对话框-->
  <el-dialog title="分配角色" :visible.sync="dialogFormVisibleRol">
    <el-form :model="form">
      <el-form-item label="用户名" label-width="100px">{{currUsername}}</el-form-item>
      <el-form-item label="角色" label-width="100px">
        <el-select v-model="currRoleId">
          <!--！！重点 ：如果select的绑定数据的值 和 option的value一样，就会显示该option的label值-->
          <el-option label="请选择" :value="-1"></el-option>
          <el-option :label="item.roleName" :value="item.id" v-for="(item,i) in roles" :key="i"></el-option>
        </el-select>
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="dialogFormVisibleRol = false">取 消</el-button>
      <el-button type="primary" @click="setRole()">确 定</el-button>
    </div>
  </el-dialog>
</el-card>
</template>

<script>
export default {
  data() {
    return {
      query: '',
      pagenum: 1,
      pagesize: 2,
      userList: [],
      total: 6,
      // 添加对话框属性
      dialogFormVisibleAdd: false,
      // 编辑对话框属性
      dialogFormVisibleEdit: false,
      dialogFormVisibleRol: false,
      // 添加用户的表单数据
      form: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 分配角色
      currRoleId: -1,
      currUserId: -1,
      currUsername: '',
      // 保存所有角色数据
      roles: []
    }
  },
  watch: {
    // 搜索功能
    query(newVal, oldVal) {
      this.$http
        .get('http://localhost:3000/data?username_like=' + newVal)
        .then((res) => {
          this.userList = res.data
        })
    }
  },
  mounted() {
    this.getUserList()
  },
  methods: {
    //! 分配角色 -发送请求  主要问题：没有修改用户的id 然后数值不能发送
    async setRole() {
      // user/:id/role
      // :id 要修改的用户的id值
      // 请求体中rid修改的新值角色id
      const res = await this.$http.put(`http://localhost:3000/data/${this.currUserId}`, {
        rid: this.currRoleId
      })
      console.log(res)
      // 关闭对话框
      this.dialogFormVisibleRol = false
    },
    // !分配角色-打开对话框  主要问题：后台数据类型不一样 string和number
    async showSetUserRoleDia(user) {
      // 用户名称
      this.currUsername = user.username
      // 给currUserId赋值
      this.currUserId = user.id
      console.log(typeof (this.currUserId))
      console.log(user.id)
      // 获取所有的角色
      const res = await this.$http.get(`http://localhost:3000/rol`)
      this.roles = res.data

      // 获取当前用户的角色id->rid
      const res1 = await this.$http.get(`http://localhost:3000/data/${user.id}`)
      this.currRoleId = res1.data.rid
      this.dialogFormVisibleRol = true
    },
    // 修改状态
    async changeMgState(user) {
      console.log(user)
      // 发送请求
      // user/:uId/state/:type
      var res = await this.$http.put(`http://localhost:3000/data/${user.id}/state/${user.mg_state}`)
    },
    // !编辑用户-发送请求
    async editUser() {
      // users/:id
      const res = await this.$http.put(`http://localhost:3000/data/${this.form.id}`, this.form)
      // console.log(res)
      // 1.关闭对话框
      this.dialogFormVisibleEdit = false
      // 2.更新视图
      this.getUserList()
    },
    // !编辑用户 - 显示对话框
    showEditUserDia(user) {
      this.form = user
      // 获取用户数据
      this.dialogFormVisibleEdit = true
    },
    // !删除用户-打开消息盒子(config)
    showDeleUserMsgBox(userId) {
      this.$confirm('删除用户?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        .then(async () => {
          // 发送请求 :id---->用户id
          // 1.data中的userId   x
          // 2.把userId以showDeleUserMsgBox参数形式传进来
          const res = await this.$http.delete(`http://localhost:3000/data/${userId}`)
          console.log(res)
          const {
            status
          } = res
          if (status === 200) {
            this.pagenum = 1
            // 更新视图
            this.getUserList()
            // 提示
            this.$message({
              type: 'success',
              message: '删除成功!'
            })
          }
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    // !添加用户功能-发送请求
    async addUser() {
      // 3.关闭对话框
      this.dialogFormVisibleAdd = false
      const res = await this.$http.post('http://localhost:3000/data', this.form)
      console.log(res)
      var {
        status
      } = res
      if (status === 201) {
        // 1.提交成功
        this.$message.success('提交成功')
        // 2.更新视图
        this.getUserList()
        // 4.清空文本框
        this.form = {}
      }
    },
    // 添加用户-显示对话框
    showAddUserDia() {
      this.form = {}
      this.dialogFormVisibleAdd = true
    },
    // 清空搜索框 重新获取数据
    loadUserList() {
      this.getUserList()
    },
    // 搜索功能
    searchUser() {
      this.getUserList()
    },
    // !获取用户列表的请求
    // !第一种方法
    getUserList(val) {
      this.$http.get(`http://localhost:3000/data?
       query=${this.query}
         &pagenum=${this.pagenum}
        &pagesize=${this.pagesize}`)
        .then((res) => {
          // console.log(res.data.length)
          var {
            status,
            data
          } = res
          // this.userList = data.splice(0, val)
          this.userList = data.splice(0, val)
          if (status === 200) {
            this.userList = data
            this.total = this.userList.length
            this.$message.success('数据获取成功')
          } else {

          }
        })
    },

    // !第二种方法
    // async getUserList() {
    //   //  uery查询参数可以为空
    //   //  pagenum当前页码不能为空
    //   //  pagesize每页显示条数不能为空
    //   var res = this.$http.get(
    //     `http://localhost:3000/data?
    //     query=${this.query}
    //     &pagenum=${this.pagenum}
    //     &pagesize=${this.pagesize}`
    //   )
    //   console.log(res)
    // }

    // !分页方法
    /* 24条pagenum=3 pagesize=2,1,2/3,4/5,6 */
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`)
      // console.log(this.userList.splice(1, val))
      this.pagesize = val
      this.getUserList(val)
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`)
    }
  }
}
</script>

<style>
.box-card {
  height: 100%;
}

.inputSearch {
  width: 300px;
}

.searchRow {
  margin-top: 20px;
}
</style>
