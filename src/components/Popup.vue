<script setup lang="ts">
import { isReactive, isRef, ref, type Ref } from "vue";
import type Employee from "@/types/employee";

const emit = defineEmits(["add", "close", "edit"]);
const props = defineProps(["editing", "employee"]);

function addEmployee() {
  emit("add");
}

function editEmployee() {
  emit("edit");
}
</script>

<template>
  <div class="popup-bg">
    <div class="popup">
      <div class="inputs">
        <input v-model="$props.employee.name" placeholder="Name" type="text" />
        <input
          v-model="$props.employee.position"
          placeholder="Position"
          type="text"
        />
        <input
          v-model="$props.employee.office"
          placeholder="Office"
          type="text"
        />
        <input v-model="$props.employee.age" placeholder="Age" type="text" />
        <input
          v-model="$props.employee.startDate"
          placeholder="Start date"
          type="text"
        />
        <input
          v-model="$props.employee.salary"
          placeholder="Salary"
          type="text"
        />
      </div>
      <div class="buttons">
        <button v-show="props.editing" @click="editEmployee">
          Edit Employee
        </button>
        <button v-show="!props.editing" @click="addEmployee">
          Add Employee
        </button>
        <button @click="$emit('close')">Close</button>
      </div>
    </div>
  </div>
</template>

<style>
input {
  margin: 3px;
}

button {
  margin: 0px 3px;
}

.inputs {
  display: flex;
  flex-direction: column;
  margin: 3px;
}

.buttons {
  display: flex;
  flex-direction: row;
  justify-content: flex-end;
  margin: 3px;
}

.popup-bg {
  position: fixed;
  background: rgba(0, 0, 0, 0.3);
  z-index: 1;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  justify-content: center;
  align-items: center;
}

.popup {
  background-color: rgba(255, 255, 255, 1);
  border-radius: 5px;
  box-shadow: 2px 2px 20px 1px;
  overflow-x: auto;
  display: flex;
  flex-direction: column;
  padding: 20px;
}
</style>
