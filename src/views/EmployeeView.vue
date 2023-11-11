<template>
  <div class="home">
      <h1>员工信息管理</h1>
      <el-form :inline="true">
        <el-form-item>
          <el-input v-model="queryForm.id"  placeholder="员工编号" class="inputVal"></el-input>
        </el-form-item>
        <el-form-item>
          <el-input v-model="queryForm.name"  placeholder="员工姓名" class="inputVal"></el-input>
        </el-form-item>
        <el-form-item>
          <el-input v-model="queryForm.dept"  placeholder="所在部门" class="inputVal"></el-input>
        </el-form-item>
        <el-form-item>
          <el-input v-model="queryForm.position"  placeholder="员工职位" class="inputVal"></el-input>
        </el-form-item>
        <el-form-item>
          <el-input v-model="queryForm.salary"  placeholder="员工薪资" class="inputVal"></el-input>
        </el-form-item>
        
        <el-button type="primary" @click="queryData">查询</el-button>

  </el-form>

  <el-row>
    <el-button type="danger" @click="delBatch">删除</el-button>
    <el-button type="primary" @click="showAddDialog()">新增</el-button>
  </el-row>
  <el-table :data="tableData" style="left: 160px; right: 100px;" @selection-change="selectionChange">
      <el-table-column type="selection"></el-table-column>
      <el-table-column label="员工编号">
        <template slot-scope="scope">
            {{ scope.row.id }}
          </template>
      </el-table-column>
      <el-table-column label="员工姓名">
        <template slot-scope="scope">
          {{scope.row.name}}
        </template>
      </el-table-column>
      <el-table-column label="所在部门">
        <template slot-scope="scope">
          {{scope.row.dept}}
        </template>
      </el-table-column>
      <el-table-column label="员工职位">
        <template slot-scope="scope">
          {{scope.row.position}}
        </template>
      </el-table-column>
      <el-table-column label="员工薪资">
        <template slot-scope="scope">
          {{scope.row.salary}}
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button type="primary" @click="showEditDialog(scope.row)">编辑</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination layout="total,sizes,prev,pager,next,jumper" 
    :total="total" :page-sizes="[5,10,15]" :page-size="pageSize" :current-page="pageNum"
    @current-change="currentPage">
    </el-pagination>

    <!-- 弹出框新增 -->
    <el-dialog :visible="visibleDialogAdd">
          <el-form>
            <el-form-item >
              <el-input v-model="addData.id" placeholder="员工编号"  ></el-input> 
              <el-input v-model="addData.name" placeholder="员工名称"   ></el-input>
              <el-input v-model="addData.dept" placeholder="所在部门"  ></el-input>
              <el-input v-model="addData.position" placeholder="职位" ></el-input>
              <el-input v-model="addData.salary" placeholder="员工薪资" ></el-input>
              <el-button type="primary" @click="addOrEdit('add')">新增</el-button>
              <el-button type="primary" @click="visibleDialogAdd=false">取消</el-button>
            </el-form-item>
          </el-form>
        </el-dialog>

        <!-- 弹出框编辑 -->
        <el-dialog :visible="visibleDialogEdit">
          <el-form>
            <el-form-item >
              <el-input v-model="editData.id" disabled placeholder="员工编号"  ></el-input> 
              <el-input v-model="editData.name" placeholder="员工名称"   ></el-input>
              <el-input v-model="editData.dept" placeholder="所在部门"  ></el-input>
              <el-input v-model="editData.position" placeholder="职位" ></el-input>
              <el-input v-model="editData.salary" placeholder="员工薪资" ></el-input>
              <el-button type="primary" @click="addOrEdit('edit')">修改</el-button>
              <el-button type="primary" @click="visibleDialogEdit=false">取消</el-button>
            </el-form-item>
          </el-form>
        </el-dialog>

  </div>
</template>

<script>

import axios from 'axios'

const BASE_API = "http://localhost:9999/employee";

export default {
  name: 'EmployeeView',
  data(){
    return {
      total:21,
      pageSize: 5,
      pageNum: 1,
      visibleDialogAdd: false,
      visibleDialogEdit: false,
      visibleDialogDel: true,
      queryForm: {"id":"","name":"","dept":"","position":"","salary":""},
      tableData: [
        {"id":"111","name":"bob","dept":"工程部","position":"讲师","salary":2500.45},
        {"id":"111","name":"bob","dept":"工程部","position":"讲师","salary":2500.45},
        {"id":"111","name":"bob","dept":"工程部","position":"讲师","salary":2500.45},
        {"id":"111","name":"bob","dept":"工程部","position":"讲师","salary":2500.45},
        {"id":"111","name":"bob","dept":"工程部","position":"讲师","salary":2500.45}
      ],
      addData:{"id":"","name":"","dept":"","position":"","salary":"","addOrEdit":"add"},
      editData:{"id":"","name":"","dept":"","position":"","addOrEdit":"edit"},
      delIds:""
    }
  },
  methods:{
    queryData(){
        axios.post(BASE_API+'/list',{pageNum:this.pageNum,pageSize:this.pageSize,body: this.queryForm}).then(resp=>{
          this.tableData=resp.data.list;
          this.total = resp.data.total;
          this.pageNum =resp.data.pageNum;
          console.log(resp.data);
        })
        .catch(error=>{
          console.log('打印错误信息'+error)
          this.tableData = [{"id":"","name":"","dept":"","position":"","salary":1200.34}];
        })
    },
    currentPage(iPage){
      this.pageNum = iPage;
      this.queryData();
    },
    showAddDialog(){
        this.visibleDialogAdd = true;

        this.addData.id = ''
        this.addData.manager = ''
        this.addData.name = ''
    },
    showEditDialog(row){
        this.editData.id = row.id;
        this.editData.manager = row.manager;
        this.editData.name = row.name;
        this.visibleDialogEdit = true;
    },
    addOrEdit(operateFlag){
      let postData = {};
      if('add'== operateFlag){
        postData = this.addData;
      }else{
        postData = this.editData;
      }
      axios.post(BASE_API+"/save", postData) //addData:{id:'',manager:'',name:''},
      .then(resp=>{ 
        console.log(resp)
        if(resp.data>0){
          this.$message('保存成功')
          if("add" == operateFlag){
            this.visibleDialogAdd = false
          }else{
            this.visibleDialogEdit = false;
          }
          this.queryData();
        }else{
          this.$message('保存失败')
          this.visibleDialogAdd=false
        }
      }) 
    },
    delBatch(){ //ids  "'001','002'" ->mybatis  delete from dept where id in ( #{id} )
      const paramData = { ids: this.delIds };
      console.log('delIds='+this.delIds)
      // axios.delete(BASE_API+"/dept/removes",{data: this.deptDel})
      axios.delete(BASE_API+"/removes",{ 
        params: paramData
      })
      .then(resp=>{ 
        if(resp.data>0){
          this.$message('删除成功') 
          this.queryData()
        }else{
          this.$message('删除失败') 
        }
      }) 
    },
    selectionChange(rows){
      let ids = '';
      rows.forEach((element) => {
        ids+=  "'"+element.id+ "',"
      });
      console.log(ids);
      this.delIds=ids.substring(0,ids.lastIndexOf(','))  //'001','002'
      // alert(ids)
      console.log("ids=",this.delIds); 
    }
  },
  mounted: function(){
      this.queryData();
  }

}

</script>

<style>
.inputVal{
  width: 150px;
}
</style>
