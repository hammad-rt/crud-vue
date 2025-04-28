<script>
// Import Axios
import axios from "axios";

export default {
  data() {
    return {
      users: [], // To store fetched users
      newUser: {
        name: "",
        email: "",
        phone: "",
      },
      editUser: null, // To store the user being edited
      apiUrl: "https://jsonplaceholder.typicode.com/users",
      errors: {}, // To store form validation errors
      loading: false, // To track API call status
    };
  },
  methods: {
    // Fetch all users
    async fetchUsers() {
      this.loading = true;
      try {
        const response = await axios.get(this.apiUrl);
        this.users = response.data;
      } catch (error) {
        console.error("Error fetching users:", error);
      } finally {
        this.loading = false;
      }
    },

    // Validate form fields
    validateForm(user) {
      this.errors = {}; // Reset errors

      if (!user.name) {
        this.errors.name = "Name is required.";
      }
      if (!user.email) {
        this.errors.email = "Email is required.";
      } else if (!/\S+@\S+\.\S+/.test(user.email)) {
        this.errors.email = "Email must be a valid email address.";
      }
      if (!user.phone) {
        this.errors.phone = "Phone number is required.";
      } else if (!/^\d{10}$/.test(user.phone)) {
        this.errors.phone = "Phone number must be a valid 10-digit number.";
      }

      return Object.keys(this.errors).length === 0; // Return true if no errors
    },

    // Create a new user
    async createUser() {
      if (!this.validateForm(this.newUser)) {
        return; // Stop if validation fails
      }

      this.loading = true;
      try {
        const response = await axios.post(this.apiUrl, this.newUser);
        this.users.push(response.data); // Add the new user to the list
        this.newUser = { name: "", email: "", phone: "" }; // Reset form
        alert("User created successfully!");
      } catch (error) {
        console.error("Error creating user:", error);
      } finally {
        this.loading = false;
      }
    },

    // Edit an existing user
    startEdit(user) {
      this.editUser = { ...user }; // Clone the user to avoid direct mutation
    },

    async updateUser() {
      if (!this.validateForm(this.editUser)) {
        return; // Stop if validation fails
      }

      this.loading = true;
      try {
        const response = await axios.put(
          `${this.apiUrl}/${this.editUser.id}`,
          this.editUser
        );
        const index = this.users.findIndex(
          (user) => user.id === this.editUser.id
        );
        if (index !== -1) {
          this.users[index] = response.data; // Update the user in the list
        }
        this.editUser = null; // Clear the edit form
        alert("User updated successfully!");
      } catch (error) {
        console.error("Error updating user:", error);
      } finally {
        this.loading = false;
      }
    },

    // Delete a user
    async deleteUser(userId) {
      this.loading = true;
      try {
        await axios.delete(`${this.apiUrl}/${userId}`);
        this.users = this.users.filter((user) => user.id !== userId); // Remove the user from the list
        alert("User deleted successfully!");
      } catch (error) {
        console.error("Error deleting user:", error);
      } finally {
        this.loading = false;
      }
    },
  },
  mounted() {
    // Fetch users when the component is mounted
    this.fetchUsers();
  },
};
</script>

<template>
  <div class="container">
    <h1>User Management</h1>

    <!-- Form to create a new user -->
    <form v-if="!editUser" @submit.prevent="createUser" class="form">
      <div class="form-group">
        <input v-model="newUser.name" placeholder="Name" class="input" />
        <span v-if="errors.name" class="error">{{ errors.name }}</span>
      </div>
      <div class="form-group">
        <input v-model="newUser.email" placeholder="Email" class="input" />
        <span v-if="errors.email" class="error">{{ errors.email }}</span>
      </div>
      <div class="form-group">
        <input v-model="newUser.phone" placeholder="Phone" class="input" />
        <span v-if="errors.phone" class="error">{{ errors.phone }}</span>
      </div>
      <button type="submit" class="btn" :disabled="loading">Add User</button>
    </form>

    <!-- Form to edit an existing user -->
    <form v-else @submit.prevent="updateUser" class="form">
      <div class="form-group">
        <input v-model="editUser.name" placeholder="Name" class="input" />
        <span v-if="errors.name" class="error">{{ errors.name }}</span>
      </div>
      <div class="form-group">
        <input v-model="editUser.email" placeholder="Email" class="input" />
        <span v-if="errors.email" class="error">{{ errors.email }}</span>
      </div>
      <div class="form-group">
        <input v-model="editUser.phone" placeholder="Phone" class="input" />
        <span v-if="errors.phone" class="error">{{ errors.phone }}</span>
      </div>
      <button type="submit" class="btn" :disabled="loading">Update User</button>
      <button
        type="button"
        @click="editUser = null"
        class="btn cancel"
        :disabled="loading"
      >
        Cancel
      </button>
    </form>

    <!-- List of users -->
    <ul class="user-list">
      <li v-for="user in users" :key="user.id" class="user-item">
        <div class="user-info">
          <strong>{{ user.name }}</strong>
          <p>Email: {{ user.email }}</p>
          <p>Phone: {{ user.phone }}</p>
        </div>
        <div class="user-actions">
          <button @click="startEdit(user)" class="btn edit" :disabled="loading">
            Edit
          </button>
          <button
            @click="deleteUser(user.id)"
            class="btn delete"
            :disabled="loading"
          >
            Delete
          </button>
        </div>
      </li>
    </ul>
  </div>
</template>

<style>
/* Add your styles here */
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.form {
  margin-bottom: 20px;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 5px;
  background-color: #f9f9f9;
}

.form-group {
  margin-bottom: 15px;
}

.input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 14px;
}

.btn {
  padding: 10px 15px;
  border: none;
  border-radius: 5px;
  font-size: 14px;
  cursor: pointer;
}

.btn:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.btn.cancel {
  background-color: #f44336;
  color: white;
}

.btn.edit {
  background-color: #2196f3;
  color: white;
}

.btn.delete {
  background-color: #f44336;
  color: white;
}

.btn {
  background-color: #4caf50;
  color: white;
}

.error {
  color: red;
  font-size: 0.9em;
}

.user-list {
  list-style: none;
  padding: 0;
}

.user-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  background-color: #fff;
}

.user-info {
  flex: 1;
}

.user-actions {
  display: flex;
  gap: 10px;
}

@media (max-width: 600px) {
  .user-item {
    flex-direction: column;
    align-items: flex-start;
  }

  .user-actions {
    margin-top: 10px;
  }
}
</style>
