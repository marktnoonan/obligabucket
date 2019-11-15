<template>
  <div class="hello">
    <h1>Obligabucket</h1>
    <p>
      Obligabucket is a like
      <mark>bucket</mark> for your
      <mark>obligations</mark> that does
      maths. To do the maths, it first needs to know two things: the size of your your bucket, and what exactly you're trying to jam in there this week. This page asks you for your
      <strong>availability</strong>, and the
      <strong>tasks</strong> you want to do,
      <strong>how long</strong> you think they will take, and
      <strong>how sure you are</strong> about how long they will take.
    </p>
    <p class="body-text">
      All data is preserved in your browser's local storage, but nowhere else. Made by Mark Noonan (
      <a href="https://twitter.com/marktnoonan">@marktnoonan</a>). Not even a version 0.
    </p>
    <hr />
    <details open>
      <summary>
        <h2>1. Your Availability</h2>
      </summary>
      <label class="total-weekly-input">
        <div>
          What is the
          <strong>total</strong> number of hours available to you over 5 days for
          <strong>non-routine</strong> tasks?
        </div>
        <input type="number" v-model="totalWeeklyHours" />
      </label>
      <h3>More Details</h3>
      <p
        class="body-text"
      >This should be a number quite a bit lower than your average work week. If you work 9-5, 5 days a week, you might feel like you have 40 hours to get stuff done, but that's unlikely to be the case. Here's how to find the real number of hours you can put against new work:</p>
      <ul class="body-text">
        <li>Start with your usual working week's number of hours. Let's say 40.</li>
        <li>Subtract necessary breaks (an hour a day seems reasonable).</li>
        <li>Pull out all recurring meetings, stand-ups, check-ins, and 1:1s. You can't schedule anything else during those times: you already know what you're doing then.</li>
        <li>Now think about friction caused by that existing recurring work. Meetings require some preparation before, and a little cooldown period after. Add about 15 minutes to every meeting just for getting in and out.</li>
        <li>Now just think about other essential work things that you have to do. This includes taking time to socialize with your coworkers during the day, as well as helping with things other people are working on, and answering questions or explaining your work to others. Maybe this is another hour in the day, maybe less, maybe more.</li>
      </ul>
      <p class="body-text">
        If you do this exercise and find you've come up with
        <i>half of your workweek is meetings, eating, using the restroom, and just being a human being in general</i> ... so be it! You have 20 hours left to do what people probably think of as "your job". As long as you don't tell people you're going to do 40 hours of work those 20 hours, things are probably gonna turn out OK.
      </p>
    </details>
    <hr />
    <details ref="taskDetailsEl" open>
      <summary>
        <h2>2. Tasks You're Committing To in the Next Five Days</h2>
      </summary>
      <p
        class="body-text"
      >Everything you can think of that you're supposed to do, at a high level. Nothing less than an hour. If there lots of real quick things you know you have to do, add them together and call that an hour.</p>
      <form @submit.prevent="addTask" class="o-form">
        <label>
          <div>Task</div>
          <input ref="newtask" required type="text" v-model="newTaskName" />
        </label>
        <label>
          <div>Hours</div>
          <input required type="number" v-model="newTaskHours" />
        </label>
        <label>
          <div>Quality of Estimate</div>
          <select type="number" v-model="newTaskQuality">
            <option value="guess">Wild Guess</option>
            <option value="ballpark">Ballpark</option>
            <option value="precise">Precise</option>
          </select>
        </label>
        <div>
          <button class="add-button">{{saveButtonText}}</button>
        </div>
      </form>
    </details>
    <hr />
    <section class="results">
      <div class="totals">
        <h2>Results</h2>
        <p class="total-possible">
          <strong>Total Hours Available</strong>
          <br />
          {{totalWeeklyHours}}
        </p>
        <p class="total-remaining">
          <strong>Total Hours Remaining</strong>
          <br />
          {{hoursRemaining}}
        </p>
        <p class="wiggle-room">
          <strong>Wiggle Room</strong>
          <br />
          {{wiggleRoom}}
        </p>
      </div>
      <div class="tasks">
        <h2>Tasks</h2>
        <p v-if="!tasks.length">None</p>
        <ol>
          <li class="results-item" v-for="task in tasks" :key="task.id">
            <div>
              {{task.name}} - {{task.quality}} - {{task.hours}} hour{{task.hours > 1 ? "s" : ""}}
              <button
                class="remove"
                @click="deleteTask(task.id)"
              >Remove</button>
              <button class="edit" @click="editTask(task.id)">Edit</button>
            </div>
            <Checklist 
              :initialList="task.checklist" 
              :parentId="task.id"
              @updateChecklist="updateCheckList"/>
          </li>
        </ol>
      </div>
    </section>
  </div>
</template>

<script>
import Checklist from "./Checklist";
export default {
  name: "Bucket",
  components: {
    Checklist
  },
  data() {
    return {
      totalWeeklyHours: 20,
      newTaskName: "",
      newTaskHours: "",
      newTaskQuality: "ballpark",
      tasks: [],
      saveButtonText: "Add",
      editing: false,
      editingId: ""
    };
  },
  methods: {
    addTask() {
      if (this.editing) {
        this.deleteTask(this.editingId);
        this.editing = false;
      }
      this.tasks.push({
        name: this.newTaskName,
        hours: this.newTaskHours,
        quality: this.newTaskQuality,
        checklist: [],
        id: (
          this.newTaskName +
          this.newTaskHours +
          this.newTaskQuality
        ).replace(" ", "")
      });
      this.resetForm();
    },
    deleteTask(id) {
      let targetIndex = this.tasks.findIndex(item => item.id == id);
      this.tasks.splice(targetIndex, 1);
    },
    editTask(id) {
      let task = this.tasks.find(item => item.id == id);
      this.editing = true;
      this.editingId = id;
      this.newTaskName = task.name;
      this.newTaskHours = task.hours;
      this.newTaskQuality = task.quality;
      this.saveButtonText = "Save";
      this.$refs.taskDetailsEl.open = true;
      this.$refs.newtask.focus();
    },

    updateCheckList({id, list}){
      console.log(id, list)
      let task = this.tasks.find(item => item.id == id);
      task.checklist = list
    },
    resetForm() {
      this.newTaskName = "";
      this.newTaskHours = "";
      this.newTaskQuality = "ballpark";
      this.saveButtonText = "Add";
    }
  },
  computed: {
    hoursRemaining() {
      if (!this.tasks.length) {
        return this.totalWeeklyHours;
      }
      let totalTaskHours = this.tasks.reduce((accumulator, currentItem) => {
        return accumulator + parseInt(currentItem.hours);
      }, 0);
      return parseInt(this.totalWeeklyHours) - totalTaskHours;
    },
    wiggleRoom() {
      if (!this.tasks.length) {
        return "🏖 Infinite. You have no tasks.";
      }
      let dailyHours = Math.round(parseInt(this.totalWeeklyHours) / 5);
      let daysOfWiggleRoom = Math.floor(this.hoursRemaining / dailyHours);
      if (daysOfWiggleRoom < 0) {
        if (this.hoursRemaining < -5) {
          return "🧙‍ ~You have entered the Realm of Fantasy~";
        }
        return "☔️ You have the opposite of wiggle room. Some things will have to take less time than you think for this to work.";
      } else if (daysOfWiggleRoom < 1) {
        if (this.hoursRemaining > 0) {
          return "⛅️ A little.";
        } else {
          return "☁️ None.";
        }
      } else if (daysOfWiggleRoom == 1) {
        return "🌤 About a day";
      } else if (daysOfWiggleRoom > 1) {
        return "☀️ Plenty";
      }
    }
  },
  watch: {
    tasks: {
      handler() {
        localStorage.setItem("tasks", JSON.stringify(this.tasks));
      },
      deep: true
    },
    totalWeeklyHours: {
      handler() {
        localStorage.setItem("totalWeeklyHours", this.totalWeeklyHours);
      }
    }
  },
  mounted() {
    if (localStorage.getItem("tasks"))
      this.tasks = JSON.parse(localStorage.getItem("tasks"));
    if (localStorage.getItem("totalWeeklyHours"))
      this.totalWeeklyHours = parseInt(
        localStorage.getItem("totalWeeklyHours")
      );
  }
};
</script>

<style>
.hello {
  width: 800px;
  max-width: 97vw;
  margin: 0 auto;
  padding: 20px;
  font-size: 20px;
  text-align: left;
}
.results {
  margin: 12px 0;
  display: flex;
}
.tasks {
  margin-left: 40px;
}
input,
select,
option,
button {
  padding: 0.5em;
  margin: 10px 0;
  margin-bottom: 16px;
}

input,
select {
  height: 50px;
  width: 160px;
}

.totals {
  width: 220px;
}

.total-weekly-input input {
  display: block;
  margin: 20px auto;
  text-align: center;
}

button {
  background-color: rgba(65, 105, 225, 0.9);
  color: #fff;
  font-weight: bold;
  padding: 0.4em 1em;
  border-radius: 10px;
  border: 1px solid rgba(0, 0, 0, 0.2);
}

button.remove,
button.edit {
  background-color: salmon;
  font-size: 14px;
  padding: 2px 6px;
  position: relative;
  top: -2px;
  margin: 0;
  margin-left: 4px;
}

button.edit {
  background-color: #fff;
  color: royalblue;
}

.add-button {
  position: absolute;
  bottom: 1px;
  height: 50px;
}

.o-form {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  padding: 5px;
}

.o-form > * {
  flex: 1;
  position: relative;
  margin: 5px;
}

hr {
  margin: 40px 0;
}

.body-text {
  font-size: 16px;
}

summary {
  position: relative;
  outline: none;
  transition: all .2s ease
}

summary::-webkit-details-marker {
  color: green;
  position: absolute;
  left: -18px;
  top: 12px;
  cursor: default;
}

summary:focus {
  outline: 1px solid rgba(0, 0, 0, 0.4);
}

summary:hover {
  background-color: rgba(0, 0, 0, 0.1);
}

summary h2,
summary h3 {
  cursor: default;
}

ol li {
  margin-bottom: 10px;
}

@media screen and (max-width: 600px) {
  .results {
    flex-direction: column;
  }
  .tasks {
    margin-left: 0;
  }
  .totals {
    width: unset;
  }
  .o-form {
    flex-direction: column;
  }
  .add-button {
    position: relative;
    bottom: unset;
    display: block;
    margin-right: 0;
    margin-left: auto;
  }
  input,
  select {
    width: 100%;
  }
  ol {
    padding-left: 12px;
  }
  ol li {
    margin-bottom: 20px;
  }
}

.results-item {
  margin-bottom: 20px;
}
</style>
