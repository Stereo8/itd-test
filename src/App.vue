<template>
  <Transition name="popup">
    <Popup
      v-show="popup"
      @close="popup = false"
      @add="addEmployee"
      @edit="editEmployee"
      :editing="editing"
      :employee="editedEmployee"
    ></Popup>
  </Transition>

  <div class="topBar">
    <button
      @click="
        popup = true;
        editing = false;
        clearInputs();
      "
    >
      Add Employee
    </button>

    <div class="pagination">
      <div>
        Entries per page:
        <input type="text" class="inputPageSize" v-model="pageSize" />
      </div>
      <button @click="if (currentPage > 0) currentPage--;">&lt;&lt;</button>
      <TransitionGroup name="pageButton">
        <button
          v-for="(page, index) in pages"
          class="pageButton"
          :class="{ activeButton: index === currentPage }"
          @click="currentPage = index"
        >
          {{ index + 1 }}
        </button></TransitionGroup
      >

      <button @click="if (currentPage < pageNum - 1) currentPage++;">
        &gt;&gt;
      </button>
    </div>
  </div>

  <table>
    <thead>
      <tr>
        <th @click="headerClick('name')">Name {{ sortStates.name }}</th>
        <th @click="headerClick('position')">
          Position {{ sortStates.position }}
        </th>
        <th @click="headerClick('office')">Office {{ sortStates.office }}</th>
        <th @click="headerClick('age')">Age {{ sortStates.age }}</th>
        <th @click="headerClick('startDate')">
          Start date {{ sortStates.startDate }}
        </th>
        <th @click="headerClick('salary')">Salary {{ sortStates.salary }}</th>
        <th>✘ ?</th>
      </tr>
    </thead>

    <TransitionGroup name="tabela" tag="tbody">
      <tr
        v-for="employee in pages[currentPage]"
        :key="employee.name"
        @dblclick="clickEdit(employee)"
      >
        <td>{{ employee.name }}</td>
        <td>{{ employee.position }}</td>
        <td>{{ employee.office }}</td>
        <td>{{ employee.age }}</td>
        <td>{{ employee.startDate }}</td>
        <td>{{ employee.salary }}</td>
        <td @click="removeEmployee(employee)">✘</td>
      </tr>
    </TransitionGroup>
  </table>
</template>

<script setup lang="ts">
import {
  ref,
  type Ref,
  TransitionGroup,
  Transition,
  reactive,
  computed,
  onMounted,
  watch,
  type ComputedRef,
} from "vue";
import { employees as arr } from "./employees";
import type Employee from "@/types/employee";
import AddEmployeePopup from "./components/Popup.vue";
import Popup from "./components/Popup.vue";

const employees: Ref<Array<Employee>> = ref(arr);
const popup = ref(false);
const editing = ref(false);

const pages: Ref<Array<Array<Employee>>> = ref([]);
const pageSize = ref(20);
const pageNum: ComputedRef<number> = computed(() =>
  Math.ceil(employees.value.length / pageSize.value)
);
const currentPage = ref(0);

let employeeToEdit: Employee;

let editedEmployee = reactive({
  name: "",
  position: "",
  office: "",
  age: "",
  startDate: "",
  salary: "",
});

let sortStates = reactive({
  name: "",
  position: "",
  office: "",
  age: "",
  startDate: "",
  salary: "",
});

function sortAsc(column: keyof Employee) {
  employees.value.sort((e1: Employee, e2: Employee) => {
    let prvi, drugi;
    if (column === "salary") {
      prvi = parseInt(e1.salary.replaceAll("$", "").replaceAll(",", ""));
      drugi = parseInt(e2.salary.replaceAll("$", "").replaceAll(",", ""));
    } else {
      prvi = e1[column].toLowerCase();
      drugi = e2[column].toLowerCase();
    }

    return prvi > drugi ? 1 : -1;
  });
}

function sortDesc(column: keyof Employee) {
  employees.value.sort((e1: Employee, e2: Employee) => {
    let prvi, drugi;
    if (column === "salary") {
      prvi = parseInt(e1.salary.replaceAll("$", "").replaceAll(",", ""));
      drugi = parseInt(e2.salary.replaceAll("$", "").replaceAll(",", ""));
    } else {
      prvi = e1[column].toLowerCase();
      drugi = e2[column].toLowerCase();
    }

    return prvi > drugi ? -1 : 1;
  });
}

function headerClick(column: keyof Employee) {
  if (sortStates[column] === "" || sortStates[column] === "▼") {
    Object.assign(sortStates, {
      name: "",
      position: "",
      office: "",
      age: "",
      startDate: "",
      salary: "",
    });
    sortStates[column] = "▲";
    sortAsc(column);
  } else if (sortStates[column] === "▲") {
    sortStates[column] = "▼";
    sortDesc(column);
  }
  paginate();
}

function addEmployee() {
  popup.value = false;
  const newEmployee: Employee = {
    name: editedEmployee.name,
    position: editedEmployee.position,
    office: editedEmployee.office,
    age: editedEmployee.age,
    startDate: editedEmployee.startDate,
    salary: editedEmployee.salary,
  };

  employees.value.push(newEmployee);

  resort();
  paginate();
}

function removeEmployee(employee: Employee) {
  employees.value = employees.value.filter((e) => {
    return e !== employee;
  });
  paginate();
}

function clickEdit(employee: Employee) {
  Object.assign(editedEmployee, employee);
  employeeToEdit = employee;
  editing.value = true;
  popup.value = true;
}

function editEmployee() {
  popup.value = false;
  const i = employees.value.findIndex((e) => e === employeeToEdit);
  console.log(editedEmployee);
  Object.assign(employees.value[i], editedEmployee);
  clearInputs();
  resort();
  paginate();
  editing.value = false;
}

function clearInputs() {
  Object.assign(editedEmployee, {
    name: "",
    position: "",
    office: "",
    age: "",
    startDate: "",
    salary: "",
  });
}

function resort() {
  for (const prop in sortStates) {
    const col = prop as keyof Employee;
    if (sortStates[col] === "▲") {
      sortAsc(col);
    } else if (sortStates[col] === "▼") {
      sortDesc(col);
    }
  }
}

function paginate() {
  pages.value = [];
  for (let i = 0; i < pageNum.value; i++) {
    let start: number = i * pageSize.value;
    let end: number = i * pageSize.value + +pageSize.value;
    const newPage = employees.value.slice(start, end);
    pages.value.push(newPage);
  }
  if (currentPage.value > pageNum.value - 1)
    currentPage.value = pageNum.value - 1;
}

onMounted(() => {
  paginate();
});

watch(pageSize, () => {
  if (pageSize.value > 2) paginate();
});
</script>

<style>
@import "./assets/base.css";

#app {
  max-width: 1280px;
  margin: 0 auto;
  padding: 2rem;

  font-weight: normal;
}

body {
  display: flex;
  place-items: center;
}

table {
  border-collapse: collapse;
  width: 100%;
}

th {
  font-weight: bold;
}

th,
td {
  text-align: left;
  padding: 3px 7px;
}

tr:nth-child(even) {
  background-color: #f2f2f2;
}

.topBar {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}

.pagination {
  display: flex;
  flex-direction: row;
}

.pageButton {
  border: 2px;
  border-radius: 3px;
  font-size: larger;
  padding: 1px 5px;
}

.activeButton {
  background-color: palegreen;
}

.inputPageSize {
  width: 40px;
  margin-right: 20px;
}

.tabela-move,
.tabela-enter-active,
.tabela-leave-active {
  transition: all 0.3s ease;
}

.tabela-enter-from,
.tabela-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

.tabela-leave-active {
  position: absolute;
}

.pageButton-move,
.pageButton-enter-active,
.pageButton-leave-active {
  transition: all 0.2s ease;
}

.pageButton-enter-from,
.pageButton-leave-to {
  opacity: 0;
  transform: translateY(20px);
}

.pageButton-leave-active {
  position: absolute;
}

.popup-enter-active,
.popup-leave-active {
  transition: opacity 0.2s ease;
}

.popup-enter-from,
.popup-leave-to {
  opacity: 0;
}
</style>
