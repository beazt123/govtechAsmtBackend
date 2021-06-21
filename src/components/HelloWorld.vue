<template>
  <div class="hello">
    <h1>Employee Dashboard</h1>
    <el-divider class="divider">Upload Employee Records</el-divider>
    <div class="upload-section">
      <el-upload
        action="action"
        :file-list="files"
        :http-request="handleUpload"
        :on-change="onFilePicked"
        :on-remove="onFileRemoved"
        accept=".csv"
        class="upload-handler"
      >
        <el-button size="small" type="info" icon="el-icon-upload">
          Click to upload
        </el-button>
      </el-upload>
      <el-button size="small" type="primary" round plain @click="submitEmployeeData">Submit</el-button>
    </div>
    <el-divider class="divider">Choose Salary Range (min - max)</el-divider>
    <div>
      <div class="min-max-holder">
        <el-slider id="salary" v-model="salaryRange" range :step="100" :min="0" :max="10000" show-tooltip :marks="salaryMarks"></el-slider>
      </div>
      
    </div>

    <el-divider class="divider">Choose Sorting Options</el-divider>
    <div>
      <el-select v-model="sortBy" placeholder="Select Sorting By">
        <el-option
          v-for="item in sortByOptions"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        ></el-option>
      </el-select>
      <el-switch class="sort-switch" v-model="sortDirection" active-text="Ascending" inactive-text="Descending"></el-switch>
    </div>
    <div class="button-holder">
      <el-button type="primary" v-on:click="searchForEmployees" round size="small">Search</el-button>
    </div>

    <el-divider class="divider">Search Employee Records</el-divider>
    <el-table
      :data="tableData"
      stripe
      size="mini"
      height="400"
      max-height="400"
      style="width: 100%">    
      <el-table-column
        prop="id"
        label="ID">
      </el-table-column>
      <el-table-column
        prop="login"
        label="Login"
        width="180">
      </el-table-column>
      <el-table-column
        prop="name"
        label="Name"
        width="180">
      </el-table-column>
      <el-table-column
        prop="salary"
        label="Salary">
      </el-table-column>
    </el-table>
    <el-pagination
        layout="prev, pager, next"
        :page-count="pageCount"
        v-model:current-page="currentPage"
        v-on:current-change="searchForEmployees">
    </el-pagination>
  </div>
</template>

<script>

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  methods:{
    handleUpload: function() {
      console.log("uploaded file")
    },
    onFilePicked: function(file, fileList){
      this.files = fileList.slice(1);
      let filename = file.name;
      console.log(filename);
      this.employeeData = file.raw;
    },
    onFileRemoved: function(file, fileList) {
      this.files = fileList;
      this.employeeData = null;
    },
    submitEmployeeData() {
      console.log("Submitting file");
      if (this.employeeData != null){
        console.log("file detected");
        console.log(this.employeeData)
        var bodyFormData = new FormData();
        bodyFormData.append('file', this.employeeData);
        // this.employeeData 
        // this.axios.get("http://localhost:5000/").then((response) => {
        //   console.log(response.data)
        // })
        this.axios({
          method: 'post',
          url: 'http://localhost:5000/users/upload',
          data: bodyFormData,
          headers: { 
            "Content-Type": "multipart/form-data", 
          },//"Access-Control-Allow-Origin": "*" 
        }).then(function (response) {
          //handle success
          console.log(response.data);
          
        }).catch( (error) => {
          //handle error
          console.log(error.response);
          this.$message.error(error.response.data.description);
        });
      }
      else {
        this.$message.error("Please attach a CSV file");
      }
    },
    searchForEmployees: function(){
      var backEndSortOperator;
      if (this.sortDirection){
        backEndSortOperator = "%2b";
      }else{
        backEndSortOperator = "%2d";
      }
      this.axios({
        method: 'get',
        url: 'http://localhost:5000/users?' 
          + 'maxSalary=' + this.salaryRange[1]
          + '&minSalary=' + this.salaryRange[0]
          + '&offset=' + ((this.currentPage - 1) * 30)
          + '&limit=' + 30
          + '&sort=' + backEndSortOperator + this.sortBy
      }).then( response => {
        this.tableData = [];
        var users = response.data.data.results;
        console.log(users);
        this.tableData = users;
      }).catch( (error) => {
        console.log(error);
        // console.log(error.response);
        // this.$message.error(error.response.data.description);
      });
    },
    printCurrentPage: function(){
      console.log(this.currentPage);
    }
  },
  data () {
    return {
      employeeData: null,
      salaryRange: [0, 10000],
      salaryMarks: {
        0: "0",
        1000: "1000",
        5000: "5000",
        10000: "10000",
      },
      pageCount: 10,
      sortByOptions: [
        {value: 'salary', label: 'Salary'},
        {value: 'name', label: 'Name'},
        {value: 'login', label: 'Login'}
      ],
      sortDirection: true,
      sortBy: "salary",
      currentPage: 1,
      tableData: [],
      files: [],
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

.min-max-holder {
  text-align: left;
  padding: 0px 40px;
  margin-bottom: 30px;
}

.upload-handler {
  margin-bottom: 10px;
}

.upload-section {
  margin-bottom: 20px;
  color: #3b3b3b;
}

.sort-switch {
  margin-left: 15px;
}

.button-holder {
  margin-top: 20px;
}

</style>
