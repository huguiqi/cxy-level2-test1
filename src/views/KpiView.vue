<template>
  <div class="home">
      <h1>绩效考核管理</h1>
      <el-form :inline="true">
        <el-form-item>
          <el-input v-model="queryForm.empId"  placeholder="员工编号" class="inputVal"></el-input>
        </el-form-item>
        <el-form-item>
          <el-input v-model="queryForm.empName"  placeholder="员工姓名" class="inputVal"></el-input>
        </el-form-item>
        <el-form-item>
          <el-input v-model="queryForm.kpi"  placeholder="当年KPI" class="inputVal"></el-input>
        </el-form-item>
        <el-button type="primary" @click="queryData">查询</el-button>

  </el-form>

  <el-table :data="tableData" style="left: 160px; right: 100px;">
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
      <el-table-column label="KPI">
        <template slot-scope="scope">
          {{scope.row.kpi}}
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button type="primary" @click="showCheckDialog(scope.row)">考核员工</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination layout="total,sizes,prev,pager,next,jumper" 
    :total="total" :page-sizes="[5,10,15]" :page-size="pageSize" :current-page="pageNum"
    @current-change="currentPage">
    </el-pagination>


        <!-- 弹出框涨薪申请 -->
        <el-dialog :visible="visibleDialogKpi">
          <el-form>
            <el-form-item >
              <el-input v-model="kpiForm.empId" disabled placeholder="员工编号" ></el-input> 
              <el-input v-model="kpiForm.empName" readonly placeholder="员工名称"></el-input>
              <el-input v-model="kpiForm.dept" readonly placeholder="所在部门"></el-input>
              <el-input v-model="kpiForm.position" readonly placeholder="职位" ></el-input>
              <el-input v-model="kpiForm.kpi" readonly placeholder="当年KPI" ></el-input>
              <el-input v-model="kpiForm.checker" placeholder="评价人"></el-input>
              <el-input v-model="kpiForm.comments" type="textarea" placeholder="评价意见"></el-input>
              <el-button type="primary" @click="checkSave()">确认</el-button>
              <el-button type="primary" @click="visibleDialogKpi=false">取消</el-button>
            </el-form-item>
          </el-form>
        </el-dialog>

  </div>
</template>

<script>

import axios from 'axios'

const BASE_API = "http://localhost:9999/kpi";

export default {
  name: 'EmployeeView',
  data(){
    return {
      total:21,
      pageSize: 5,
      pageNum: 1,
      visibleDialogKpi: false,
      queryForm: {"empId":"","empName":"","kpi":""},
      kpiForm:{"empId":"","empName":"","dept":"","position":"","kpi":"","checker":"","comments":""},
      tableData: [
        {"id":"1","name":"刘光启","dept":"销售部","position":"讲师","checker":"王大毛","comments":"有功劳","kpi":"70"},
        {"id":"2","name":"刘光启","dept":"销售部","position":"讲师","checker":"王大毛","comments":"有功劳","kpi":"30"},
        {"id":"3","name":"刘光启","dept":"销售部","position":"讲师","checker":"王大毛","comments":"有功劳","kpi":"30"},
        {"id":"4","name":"刘光启","dept":"销售部","position":"讲师","checker":"王大毛","comments":"有功劳","kpi":"30"},
        {"id":"5","name":"刘光启","dept":"销售部","position":"讲师","checker":"王大毛","comments":"有功劳","kpi":"30"}
      ]
    }
  },
  methods:{
    queryData(){
        axios.post('http://localhost:9999/employee/list',{pageNum:this.pageNum,pageSize:this.pageSize,body: this.queryForm}).then(resp=>{
          this.tableData=resp.data.list;
          this.total = resp.data.total;
          this.pageNum =resp.data.pageNum;
          console.log(resp.data);
        })
        .catch(error=>{
          console.log('打印错误信息'+error)
          // this.tableData = [{"id":"","name":"","dept":"","position":"","salary":1200.34}];
        })
    },
    currentPage(iPage){
      this.pageNum = iPage;
      this.queryData();
    },
    showCheckDialog(row){

      this.kpiForm.empId = row.empId;
      this.kpiForm.empName = row.empName;
      this.kpiForm.dept = row.dept;
      this.kpiForm.position = row.position;
      this.kpiForm.checker = row.manager;
      this.kpiForm.comments = row.comments;

      this.visibleDialogKpi = true;

      this.queryDeptManager(row.dept);
    },
    checkSave(){

      axios.post(BASE_API+"/checkSave", this.kpiForm) //addData:{id:'',manager:'',name:''},
      .then(resp=>{ 
        console.log(resp)
        if(resp.data>0){
          this.$message('提交成功')
          
        }else{
          this.$message('申请失败')
        }
        this.visibleDialogKpi = false;
        this.queryData();
      }).catch(error=>{
          console.log('打印异常日志'+error)
      }); 
    },queryDeptManager(name){
      axios.get('http://localhost:9999/dept/name',{params: {"name": name}})
          .then(resp=>{
            console.log('查询部门为:'+resp.data)
            this.kpiForm.checker = resp.data;
          }).catch(error=>{
              console.log('打印异常日志:'+error);
          })
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
