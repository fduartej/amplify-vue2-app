<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
    <amplify-authenticator>
      <h1>Employee App</h1>
      <input type="text" v-model="name" placeholder="Employee name">
      <input type="text" v-model="description" placeholder="Employee description">
      <button v-on:click="createEmployee">Create Employee</button>
      <div v-for="item in employees" :key="item.id">
        <h3>{{ item.name }}</h3>
        <p>{{ item.description }}</p>
      </div>

      <div id="amplify-signout">
        <amplify-sign-out></amplify-sign-out>
      </div>
    </amplify-authenticator>

  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import { API } from "aws-amplify";
import { listEmployees } from "./graphql/queries";
import { createEmployee } from './graphql/mutations';
import { onCreateEmployee } from './graphql/subscriptions';


export default {
  name: 'App',
  components: {
    HelloWorld
  },
  async created() {
    this.getEmployees();
    this.subscribe();
  },
  data() {
    return {
      name: '',
      description: '',
      employees: []
    }
  },
  methods: {
    async createEmployee() {
      const { name, description } = this;
      if (!name || !description) return;
      const employee = { name, description };
      await API.graphql({
        query: createEmployee,
        variables: {input: employee},
      });
      this.name = '';
      this.description = '';
    },
    async getEmployees() {
      const employees = await API.graphql({
        query: listEmployees
      });
      this.employees = employees.data.listEmployees.items;
    },
    subscribe() {
      API.graphql({ query: onCreateEmployee })
        .subscribe({
          next: (eventData) => {
            let employee = eventData.value.data.onCreateEmployee;
            if (this.employees.some(item => item.name === employee.name)) return; // remove duplications
            this.employees = [...this.employees, employee];
          }
        });
    }    
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
