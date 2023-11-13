<template>
  <div class="home" style="width: 80%;">
    <h2>部门信息管理</h2>
    <el-form :inline="true">
        <el-form-item>
          <el-input v-model="queryForm.id"  placeholder="部门编号" class="inputVal"></el-input>
        </el-form-item>
        <el-form-item>
          <el-input v-model="queryForm.manager"  placeholder="部门经理" class="inputVal"></el-input>
        </el-form-item>
        <el-form-item>
          <el-input v-model="queryForm.name"  placeholder="部门名字" class="inputVal"></el-input>
        </el-form-item>
        
        <el-button type="primary" @click="queryData">查询</el-button>

  </el-form>

  <el-row>
    <el-button type="danger" @click="delBatch">删除</el-button>
    <el-button type="primary" @click="showAddDialog()">新增</el-button>
  </el-row>
  <el-table :data="tableData" style="left: 160px; right: 100px;" @selection-change="selectionChange">
      <el-table-column type="selection"></el-table-column>
      <el-table-column label="部门编号">
        <template slot-scope="scope">
            {{ scope.row.id }}
          </template>
      </el-table-column>
      <el-table-column label="部门经理">
        <template slot-scope="scope">
          {{scope.row.manager}}
        </template>
      </el-table-column>
      <el-table-column label="部门名字">
        <template slot-scope="scope">
          {{scope.row.name}}
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
          <el-input v-model="deptAdd.id" placeholder="部门编号"  ></el-input> 
          <el-input v-model="deptAdd.manager" placeholder="部门经理"  ></el-input>
          <el-input v-model="deptAdd.name" placeholder="部门名称"   ></el-input>
          <el-button type="primary" @click="addOrEdit('add')">新增</el-button>
          <el-button type="primary" @click="visibleDialogAdd=false">取消</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>

    <!-- 弹出框编辑 -->
    <el-dialog :visible="visibleDialogEdit">
      <el-form>
        <el-form-item >
          <el-input v-model="deptEdit.id" readonly="true" placeholder="部门编号"></el-input> 
          <el-input v-model="deptEdit.manager" placeholder="部门经理"  ></el-input>
          <el-input v-model="deptEdit.name" placeholder="部门名称"   ></el-input>
          <el-button type="primary" @click="addOrEdit('edit')">修改</el-button>
          <el-button type="primary" @click="visibleDialogEdit=false">取消</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>




  </div>
</template>
<script>

import axios from 'axios'
import 'element-ui/lib/theme-chalk/index.css'

const BASE_API = "http://localhost:9999/dept";

export default {
  name: 'DeptView',
  data(){
    return {
      total:21,
      pageSize: 5,
      pageNum: 1,
      visibleDialogAdd: false,
      visibleDialogEdit: false,
      visibleDialogDel: true,
      queryForm: {"id":"","manager":"","name":""},
      tableData: [{"id":"111","manager":"bob","name":"baba"},
      {"id":"113","manager":"sam","name":"sam222"},
      {"id":"113","manager":"sam","name":"sam222"},
      {"id":"113","manager":"sam","name":"sam222"},
      {"id":"113","manager":"sam","name":"sam222"},
      {"id":"113","manager":"sam","name":"sam222"},
      {"id":"113","manager":"sam","name":"sam222"},
      {"id":"113","manager":"sam","name":"sam222"},
      {"id":"113","manager":"sam","name":"sam222"},
      {"id":"122","manager":"jake","name":"jake"}],
      deptAdd:{"id":"","manager":"","name":"","addOrEdit":"add"},
      deptEdit:{"id":"","manager":"","name":"","addOrEdit":"edit"},
      deptDel:{"ids":[]},
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
    },
    currentPage(iPage){
      this.pageNum = iPage;
      this.queryData();
    },
    showAddDialog(){
        this.visibleDialogAdd = true;

        this.deptAdd.id = ''
        this.deptAdd.manager = ''
        this.deptAdd.name = ''
    },
    showEditDialog(row){
        this.deptEdit.id = row.id;
        this.deptEdit.manager = row.manager;
        this.deptEdit.name = row.name;
        this.visibleDialogEdit = true;
    },
    addOrEdit(operateFlag){
      let postData = {};
      if('add'== operateFlag){
        postData = this.deptAdd;
      }else{
        postData = this.deptEdit;
      }
      axios.post(BASE_API+"/save", postData) //deptAdd:{id:'',manager:'',name:''},
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
      rows.forEach((element,index) => {
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
