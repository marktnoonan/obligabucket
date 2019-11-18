<template>
  <details>
    <summary>
      <h3 class="checklist-title">Checklist</h3>
    </summary>
    <ul class="checklist">
      <li v-for="item in list" :key="item.text">
        <label :class="{'completed': item.completed}">
          <input type="checkbox" :checked="item.completed" v-model="item.completed"/>
          {{item.text}}
        </label>
        <button class="remove" @click="deleteItem(item.text)">Remove</button>
        <button class="edit" @click="editItem(item.text)">Edit</button>
      </li>
    </ul>
    <form @submit.prevent="addItem">
      <input type="text" v-model="newItemText" ref="newitem" />
      <button class="edit">{{editing ? "Save" : "Add"}}</button>
    </form>
  </details>
</template>

<script>
export default {
  name: "Checklist",
  props: ["initialList", "parentId"],
  data() {
    return {
      list: [
      ],
      newItemText: "",
      editing: false
    };
  },
  mounted() {
    if (this.initialList) {
      this.list = this.initialList;
    }
  },
  methods: {
    addItem() {
      this.list.push({
        text: this.newItemText,
        completed: false
      });
      this.newItemText = "";
      this.editing = false;
      this.$emit("updateChecklist", { id: this.parentId, list: this.list });
    },
    deleteItem(text) {
      let targetIndex = this.list.findIndex(item => item.text == text);
      this.list.splice(targetIndex, 1);
      this.$emit("updateChecklist", { id: this.parentId, list: this.list });
    },
    editItem(text) {
      this.editing = true;
      let targetIndex = this.list.findIndex(item => item.text == text);
      this.list.splice(targetIndex, 1);
      this.newItemText = text;
      this.$refs.newitem.focus();
    }
  }
};
</script>

<style scoped>
input[type="checkbox"] {
  height: 20px;
  width: 20px;
  position: relative;
  top: 3px;
}
input[type="text"] {
  height: 20px;
  width: calc(100% - 100px);
  margin-left: 26px;
}
.checklist-title {
  margin: 0;
}
.checklist {
  list-style-type: none;
  padding-left: 0;
}
.checklist li {
  margin: 0;
}

summary {
  padding: 2px;
  padding-left: 6px;
}

summary::-webkit-details-marker {
  color: green;
  position: absolute;
  left: -16px;
  top: 4px;
}

details {
  margin-top: 10px;
  background-color: lavender;
  padding: 6px 20px 4px;
  border-radius: 9px;
  border: 1px solid rgba(0,0,0,0.2);
}

details h3 {
  font-size: 1rem;
  font-weight: normal;
}

.completed {
  text-decoration: line-through;
}
</style>