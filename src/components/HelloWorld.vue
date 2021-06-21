<template>
  <div class="hello">
    <h1>Employee Dashboard</h1>
    <div>
      <input type="file" id="myFile" name="filename" accept=".csv" v-on:change="onFilePicked">
      <button v-on:click="submitEmployeeData">Submit</button>
    </div>
    <div>
      <div class="min-max-holder">
        <el-input id="maxSalary" placeholder="Maximum salary" v-model="maxSalary" size="small"></el-input>
        <el-input id="minSalary" placeholder="Minimum salary" v-model="minSalary" size="small"></el-input>
      </div>
      <label for="sortBy">Sort by:</label>
      <select id="sortBy" name="parmeters" v-model="sortBy">
        <option value="id">ID</option>
        <option value="login">Login</option>
        <option value="name">Name</option>
        <option value="salary">Salary</option>
      </select>
      <input type="submit" value="Search" v-on:click="searchForEmployees">
    </div>
    <el-pagination
        layout="prev, pager, next"
        :total="50"
        v-model:current-page="currentPage"
        v-on:current-change="printCurrentPage">
    </el-pagination>
    <el-table
    :data="tableData"
    stripe
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
  </div>
</template>

<script>

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  methods:{
    onFilePicked: function(event){
      console.log("Submitted file");
      const files = event.target.files;
      let filename = files[0].name;
      console.log(filename);
      this.employeeData = files[0];
    },

    submitEmployeeData: () => {
      console.log("Uploading file");
      if (this.employeeData != null){
        console.log("file detected");
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
          headers: { "Content-Type": "multipart/form-data"},//"Access-Control-Allow-Origin": "*" 
        }).then(function (response) {
          //handle success
          console.log(response.data);
          
        }).catch( (error) => {
          //handle error
          console.log(error.response);
          this.$message.error(error.response.data.description);
        });
        
      }else{
        this.$message.message("Please attach a CSV file");
      }
    },
    searchForEmployees: function(){
      this.axios({
        method: 'get',
        url: 'http://localhost:5000/users?' 
          + 'maxSalary=' + this.maxSalary
          + '&minSalary=' + this.minSalary
          + '&offset=' + ((this.currentPage - 1) * 30)
          + '&limit=' + 30
          + '&sort=' + "%2b" + this.sortBy
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
      maxSalary: 0,
      minSalary: 0,
      sortBy: null,
      currentPage: 1,
      tableData: []
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
  display: flex;
  justify-content: space-evenly;
}
</style>
