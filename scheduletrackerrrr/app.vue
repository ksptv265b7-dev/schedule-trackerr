<template>
  <div class="min-h-screen bg-gray-100 flex items-center justify-center">
    <div v-if="!currentUser" class="w-full max-w-md bg-white shadow-lg rounded-lg p-6">
      <h2 class="text-2xl font-bold text-center text-blue-600 mb-4">MPDC Portal</h2>

      <div v-if="!showRegister">
        <h3 class="text-lg font-semibold mb-2">Login</h3>
        <form @submit.prevent="loginUser">
          <input v-model="loginUsername" type="text" placeholder="Username" class="input" required>
          <input v-model="loginPassword" type="password" placeholder="Password" class="input" required>
          <button class="btn w-full">Login</button>
        </form>
        <p class="text-center mt-3 text-sm">No account? 
          <span @click="showRegister = true" class="text-blue-500 cursor-pointer">Register here</span>
        </p>
      </div>

      <div v-else>
        <h3 class="text-lg font-semibold mb-2">Register</h3>
        <form @submit.prevent="registerUser">
          <input v-model="regUsername" type="text" placeholder="Username" class="input" required>
          <input v-model="regPassword" type="password" placeholder="Password" class="input" required>
          <button class="btn w-full">Register</button>
        </form>
        <p class="text-center mt-3 text-sm">Already have an account? 
          <span @click="showRegister = false" class="text-blue-500 cursor-pointer">Login here</span>
        </p>
      </div>
    </div>

    <div v-else class="w-full max-w-2xl bg-white shadow-lg rounded-lg p-6">
      <div class="flex justify-between items-center mb-4">
        <h2 class="text-2xl font-bold text-blue-600">MPDC Schedule Tracker</h2>
        <button @click="logout" class="bg-red-500 text-white px-3 py-1 rounded">Logout</button>
      </div>

      <form @submit.prevent="addSchedule" class="flex flex-col sm:flex-row gap-2 mb-4">
        <input v-model="newTitle" type="text" placeholder="Schedule Title" class="input flex-grow" required>
        <input v-model="newDate" type="date" class="input" required>
        <button class="btn">Add</button>
      </form>

      <div v-if="schedules.length === 0" class="text-gray-500 text-center py-4">
        No schedules yet. Add one above!
      </div>

      <ul>
        <li v-for="(s, index) in schedules" :key="index" class="flex justify-between items-center border-b py-2">
          <div>
            <span class="font-semibold">{{ s.title }}</span>
            <small class="text-gray-600 block">{{ formatDate(s.date) }}</small>
          </div>
          <button @click="removeSchedule(index)" class="text-red-500 hover:underline">Remove</button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      users: JSON.parse(localStorage.getItem("users")) || [],
      currentUser: JSON.parse(localStorage.getItem("currentUser")) || null,
      showRegister: false,
      loginUsername: "",
      loginPassword: "",
      regUsername: "",
      regPassword: "",
      newTitle: "",
      newDate: "",
      schedules: []
    };
  },
  created() {
    if (this.currentUser) this.loadSchedules();
  },
  methods: {
    saveUsers() {
      localStorage.setItem("users", JSON.stringify(this.users));
    },
    saveCurrentUser() {
      localStorage.setItem("currentUser", JSON.stringify(this.currentUser));
    },
    saveSchedules() {
      if (this.currentUser) {
        localStorage.setItem(`schedules_${this.currentUser.username}`, JSON.stringify(this.schedules));
      }
    },
    loadSchedules() {
      if (this.currentUser) {
        this.schedules = JSON.parse(localStorage.getItem(`schedules_${this.currentUser.username}`)) || [];
      }
    },
    registerUser() {
      const existing = this.users.find(u => u.username === this.regUsername);
      if (existing) return alert("Username already exists!");
      this.users.push({ username: this.regUsername, password: this.regPassword });
      this.saveUsers();
      alert("Registered successfully!");
      this.showRegister = false;
      this.regUsername = this.regPassword = "";
    },
    loginUser() {
      const user = this.users.find(
        u => u.username === this.loginUsername && u.password === this.loginPassword
      );
      if (!user) return alert("Invalid username or password!");
      this.currentUser = user;
      this.saveCurrentUser();
      this.loadSchedules();
    },
    logout() {
      this.currentUser = null;
      localStorage.removeItem("currentUser");
    },
    addSchedule() {
      this.schedules.push({ title: this.newTitle, date: this.newDate });
      this.newTitle = this.newDate = "";
      this.saveSchedules();
    },
    removeSchedule(index) {
      this.schedules.splice(index, 1);
      this.saveSchedules();
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString();
    }
  }
};
</script>

<style scoped>
.input {
  @apply border border-gray-300 rounded px-3 py-2 w-full;
}
.btn {
  @apply bg-blue-600 hover:bg-blue-700 text-white font-semibold rounded px-4 py-2;
}
</style>
