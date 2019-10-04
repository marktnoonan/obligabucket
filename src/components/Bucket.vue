<template>
  <div class="hello">
    <h1>Obligabucket</h1>
    <section class="summary">
      <div class="totals">
        <p class="total-possible">
          Total Hours Available: {{totalWeeklyHours}}
        </p>
        <p class="total-used">
          Total Hours Used: {{totalWeeklyHours}}
        </p>
      </div>
    </section>
    <form class="o-form">
      <label>
        <div>Average hours available to you over 5 days for <em>non-routine</em> tasks:</div>
      <input type="number" v-model="totalWeeklyHours" name="total-weekly-hours" id="total-weekly-hours">
      </label>
    </form>
  </div>
</template>

<script>
export default {
  name: 'Bucket',
  data() {
    return {
      totalWeeklyHours: "20"
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
.hello {
  max-width: 400px;
  margin: 0 auto;
  font-size: 20px;
  text-align: left;
}
.totals {
  outline: 1px solid;
  margin: 30px;
  padding: 20px;
}

input {
  padding: .5em;
  margin: 10px 0;
}


</style>
