<template>
  <div id="app">
    <!--<div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link>
    </div>-->
    <router-view/>
  </div>
</template>


<script>


export default {
  name: 'Home',
  data () {
    return {
      data: null,
      results0: null
    }
  },
  mounted() {
    this.getData();
  },
  methods: {
        getData() {
            return this.$http.get('https://tedyin.com/archive/avax-stakers-api')
                .then(response => {
                    this.data = response.data;
                    this.results0 = this.data.results[0].stakers;
                    this.results0.forEach(item => {
                        item.show = false
                    });
                })
                .catch(error => console.log(error))
        }
    },
}
</script>

<style>
.nodeID {
    font-family: monospace;
}
.owner {
    font-family: monospace;
}
</style>
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
