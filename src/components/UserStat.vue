<template>

<div>

  <el-button type="primary" @click="openUser('ruleform1')">添加</el-button>

 <el-dialog title="添加用户" :visible.sync="dialogFormVisible" center>

  <el-form :model="ruleform1" status-icon :rules="rules" ref="ruleform1">

    <el-form-item label="用户名称" :label-width="formLabelWidth" prop="name">
      <el-input v-model="ruleform1.name" auto-complete="off"></el-input>
    </el-form-item>

    <el-form-item label="用户邮箱" :label-width="formLabelWidth"  prop="email">
      <el-input v-model="ruleform1.email" auto-complete="off"></el-input>
    </el-form-item>
    <template v-if="!edit">
        <el-form-item label="用户密码" :label-width="formLabelWidth" prop="password">
          <el-input type="password" v-model="ruleform1.password" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="确认密码" :label-width="formLabelWidth" prop="password_confirmation" >
          <el-input type="password" v-model="ruleform1.password_confirmation" auto-complete="off"></el-input>
        </el-form-item>
    </template>
  </el-form>

  <div slot="footer" class="dialog-footer">
    <el-button @click="dialogFormVisible = false">取 消</el-button>
    <el-button type="primary" @click="submitForm('ruleform1')">确 定</el-button>
  </div>

</el-dialog>


    <el-table
    :data="tableData"
    style="width: 100%"
    @selection-change="handleSelectionChange"
    :default-sort = "{prop: 'created_at', order: 'descending'}">

    <el-table-column type="expand">
      <template slot-scope="props">
        <el-form label-position="left" inline class="demo-table-expand">
          <el-form-item label="创建日期">
            <span>{{ props.row.created_at }}</span>
          </el-form-item>
          <el-form-item label="更新日期">
            <span>{{ props.row.updated_at }}</span>
          </el-form-item>
          <el-form-item label="姓名">
            <span>{{ props.row.name }}</span>
          </el-form-item>
          <el-form-item label="邮箱">
            <span>{{ props.row.email }}</span>
          </el-form-item>
        </el-form>
      </template>
    </el-table-column>

    <el-table-column
      type="selection"
      width="55">
    </el-table-column>
    <el-table-column prop="id" label="编号" sortable></el-table-column>
      <el-table-column prop="created_at" label="创建日期" sortable></el-table-column>

      <el-table-column prop="updated_at" label="更新日期" sortable></el-table-column>

      <el-table-column prop="name" label="姓名" ></el-table-column>

      <el-table-column prop="email" label="邮箱"  sortable></el-table-column>
    <el-table-column
      fixed="right"
      label="操作">
      <template slot-scope="scope">
        <el-button
          size="mini"
          type="success"
          @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
        <el-button
          size="mini"
          type="danger"
          @click="handleDelete(scope.$index, scope.row)">删除</el-button>
      </template>
    </el-table-column>
    </el-table>

    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[10, 20, 30, 40]"
      :page-size="pagesize"
      layout="total, sizes, pager, jumper"
      :total="totalCount">
    </el-pagination>
</div>

</template>

<script>
export default {
  data() {
    var validatePass2 = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请再次输入密码"));
      } else if (value !== this.ruleform1.password_confirmation) {
        callback(new Error("两次输入密码不一致!"));
      } else {
        callback();
      }
    };

    return {
      tableData: [],
      //请求的URL
      url: "/api/admin/user",
      //默认每页数据量
      pagesize: 10,
      //当前页码
      currentPage: 1,
      //默认数据总数
      totalCount: 0,

      dialogFormVisible: false,

      ruleform1: {
        name: "",
        email: "",
        password: "",
        password_confirmation: ""
      },
      rules: {
        name: [
          { required: true, message: "请输入用户名称", trigger: "blur" },
          { min: 6, max: 30, message: "长度在 6 到 30 个字符", trigger: "blur" }
        ],
        email: [
          {
            type: "email",
            required: true,
            message: "请输入用户邮箱",
            trigger: "blur"
          }
        ],
        password: [
          { required: true, message: "请输入用户密码", trigger: "blur" }
        ],
        password_confirmation: [{ validator: validatePass2, trigger: "blur" }]
      },
      formLabelWidth: "120px",

      multipleSelection: [],
      errors: false,
      edit: false
    };
  },
  mounted: function() {},
  created: function() {
    this.getData();
  },
  methods: {
    getData: function() {
      this.$http
        .get(this.url, {
          params: { page: this.currentPage, per_page: this.pagesize }
        })
        .then(res => {
          this.tableData = res.body.data || [];

          this.totalCount = res.body.total;
        });
    },
    handleEdit(index, row) {
      console.log(index, row);
      this.edit = true;
      this.dialogFormVisible = true;
      _.extend(this.ruleform1, row);
    },
    handleDelete(index, row) {
      console.log(index, row);
      this.$confirm("是否确认删除?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http
            .delete("/api/admin/user/" + row.id, {
              emulateJSON: true
            })
            .then(
              function(res) {
                this.getData();
                this.$message({
                  type: "success",
                  message: "删除成功!"
                });
              },
              function() {
                console.log("failed");
              }
            );
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    //每页显示数据量变更
    handleSizeChange: function(val) {
      this.pagesize = val;
      this.getData();
    },

    //页码变更
    handleCurrentChange: function(val) {
      this.currentPage = val;
      this.getData();
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then(_ => {
          done();
        })
        .catch(_ => {});
    },
    submitForm: function(formName) {
      let method, url;
      if (this.edit) {
        method = "put";
        url = "/api/admin/user/" + this.ruleform1.id;
      } else {
        method = "post";
        url = "/api/admin/user";
      }
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.$http[method](url, this.ruleform1, { emulateJSON: true })
            .then(res => {
              this.$message({
                type: "success",
                message: "保存成功!"
              });
              this.getData();
              this.dialogFormVisible = false;
            })
            .catch(res => {
              this.$refs[formName].fields.forEach(field => {
                let err = "success",
                  msg = "";
                if (res.body[field.prop]) {
                  err = "error";
                  msg = res.body[field.prop][0];
                  this.errors = true;
                }
                field.validateState = err;
                field.validateMessage = msg;
              });
            });
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
    resetForm: function(formName) {
      this.$refs[formName].resetFields();
    },
    openUser: function(formName) {
      this.edit = false;
      this.ruleform1 = {
        name: "",
        email: "",
        password: "",
        password_confirmation: ""
      };
      this.dialogFormVisible = true;
    }
  }
};
</script>
