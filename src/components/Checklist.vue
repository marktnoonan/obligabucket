<template>
  <details>
    <summary>
      <h3 class="checklist-title">Checklist</h3>
    </summary>
    <ul class="checklist" ref="checklist">
      <li v-for="item in list" :key="item.text">
        <label :class="{ completed: item.completed }">
          <input
            type="checkbox"
            :checked="item.completed"
            v-model="item.completed"
          />
          {{ item.text }}
          <a v-if="item.link" :href="item.link" target="_blank">🔗</a>
        </label>
        <button
          v-if="showButtons"
          class="remove"
          @click="deleteItem(item.text)"
        >
          Remove
        </button>
        <button v-if="showButtons" class="edit" @click="editItem(item.text)">
          Edit
        </button>
      </li>
    </ul>
    <form
      v-show="addingNewItem"
      class="add-item-form"
      @submit.prevent="addItem"
    >
      <label>
        New item text
        <input
          type="text"
          class="item-text-input"
          v-model="newItemText"
          ref="newitem"
        />
      </label>
      <label>
        New item link (optional)
        <input
          type="text"
          class="item-link-input"
          v-model="newItemLink"
          ref="newitem"
        />
      </label>
      <button class="edit">Save</button>
    </form>
    <button v-show="!addingNewItem" class="edit" @click="addingNewItem = true">
      + New Item</button
    ><br />
    <hr class="small-hr" />
    <button class="edit" @click="toggleButtons">
      {{ showButtons ? "Hide" : "Show" }} Buttons
    </button>
    <button class="edit" v-clipboard="clipboardContent">
      Copy List as Text
    </button>
  </details>
</template>

<script>
export default {
  name: "Checklist",
  props: ["initialList", "parentId"],
  data() {
    return {
      list: [],
      newItemText: "",
      newItemLink: "",
      editing: false,
      showButtons: true,
      addingNewItem: false,
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
        link: this.newItemLink,
        completed: false,
      });
      this.newItemText = "";
      this.newItemLink = "";
      this.editing = false;
      this.addingNewItem = false;
      this.$emit("updateChecklist", { id: this.parentId, list: this.list });
    },
    deleteItem(text) {
      let targetIndex = this.list.findIndex((item) => item.text == text);
      this.list.splice(targetIndex, 1);
      this.$emit("updateChecklist", { id: this.parentId, list: this.list });
    },
    editItem(text) {
      this.editing = true;
      let targetIndex = this.list.findIndex((item) => item.text == text);
      this.list.splice(targetIndex, 1);
      this.newItemText = text;
      this.$refs.newitem.focus();
    },
    clipboardContent() {
      return this.list.map((item) => item.text).join("\n");
    },
    toggleButtons() {
      this.showButtons = !this.showButtons;
    },
  },
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
  height: 30px;
  width: 100%;
  margin: 0;
  display: block;
  margin-bottom: 8px;
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
  border-radius: 10px;
  left: -10px;
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
  border: 1px solid rgba(0, 0, 0, 0.2);
}

details h3 {
  font-size: 1rem;
  font-weight: normal;
}

.completed {
  text-decoration: line-through;
}

.add-item-form {
  font-size: 14px;
}

.small-hr {
  margin: 10px 0;
  border-color: rgba(0, 0, 0, 0.2);
}
</style>