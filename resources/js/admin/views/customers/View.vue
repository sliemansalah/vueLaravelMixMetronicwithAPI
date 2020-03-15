<template>
  <div>
	<h3 class="m-subheader__title ">Dashboard</h3>
	<d-t 
      title="Customers" 
      addBtn="New Customer" 
      addBtnBgColor="#00f"
      addBtnColor="#fff"
      :rows="rows" 
      :details="details"
      @add="add"
      @edit="edit" 
      @del="del">
  </d-t>
  <customer-form 
                  ref="customerForm" 
                  :customerFormStatus="customerFormStatus"
                  @save="addNewCustomer"
                  @update="updateCustomer"></customer-form>
  </div>
</template>
<script>
import axios from 'axios';
var _ = require('lodash');
export default {
  data(){
    return {
      customerFormStatus:'add',
      customers:[],
      customerID:null,
      local:false,
      rows:[
      {id:'id',value:'ID'},
      {id:'name',value:'Name'},
      {id:'age',value:'Age'},
      {id:'email',value:'Email'},
      {id:'skills',value:'Skills'},
    ],
    details:[],
    }
  },
  methods:{
    getCustomers(){
      axios.get('/api/customers').then(res=> {
      this.customers =  res.data.data
      console.log(this.customers);
      this.customers.forEach(data => {
        delete data.skills;
        delete data.created_at;
        delete data.updated_at;
        data.jsonSkills = this.skillsFilter(data.jsonSkills);
      });
      this.details = this.customers
    })
    },
    edit(res){
    this.customerFormStatus = 'edit'
    this.$refs.customerForm.clear()
    this.fillCustomer(res[0]);
    this.customerID = res[0].id;
    },
    fillCustomer(res){
    this.$refs.customerForm.customer.name = res.name
    this.$refs.customerForm.customer.email = res.email
    this.$refs.customerForm.customer.age = res.age
    if(res.jsonSkills.includes("angular")){
          this.$refs.customerForm.customer.skills.angular = true
    }
    if(res.jsonSkills.includes("vue")){
          this.$refs.customerForm.customer.skills.vue = true
    }
    if(res.jsonSkills.includes("react")){
          this.$refs.customerForm.customer.skills.react = true
    }
    },
    del(index){
      let id  = this.customers[index].id;
       swal({
                title: "Delete Customer",
                text: "Are you need delete customer",
                icon: "success",
                confirmButtonText: "<span>Yes</span>",
                showCancelButton: !0,
                cancelButtonText: "<span>No, thanks</span>",
            }).then(result => {
              if(result.value){
                this.customers.splice(index,1)
                axios.delete('/api/customers/'+id).then(res=> {
                this.details = this.customers
                swal("Customer Remove", "Customer was removed successfully!", "info")
                 this.$store.dispatch('initCustomers',this.customers)
                });
              }
            })
    },
    add(){
     this.customerID = null;
     this.customerFormStatus = 'add'
     this.$refs.customerForm.clear()
    },
    addNewCustomer(data){
       let customer_add = {
          name:data.name,
          age:data.age,
          email:data.email,
          skills:data.skills
        };
        this.customers.push(customer_add)
         axios.post(`/api/customers`, customer_add)
    .then(response => {
          swal("Add Customer!", "Customer Added Successfully!", "success")
          this.getCustomers();
    })
         
    },
    updateCustomer(data){
      console.log(this.customerID);
        let customer_add = {
          name:data.name,
          age:data.age,
          email:data.email,
          skills:data.skills
        };

          axios.put(`/api/customers/`+this.customerID, customer_add)
    .then(response => {
          swal("Update Customer!", "Customer Updated Successfully!", "success")
          this.getCustomers();
    })
    },
    skillsFilter(skills){
      let data="";
  _.forEach(skills, function(value,key) {
    if(value){
      data+=key +"/"
    }
});
data=data.substring(0,data.length -1)
    return data
    }
  },
  mounted(){
  this.getCustomers();   
  }
}
</script>

<style>

</style>
