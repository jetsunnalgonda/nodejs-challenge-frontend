<template>
    <div class="login-container">
        <h2>Login</h2>
        <form @submit.prevent="login">
            <div class="form-group">
                <label for="email">Email:</label>
                <input v-model="form.email" id="email" type="email" required />
            </div>
            <div class="form-group">
                <label for="password">Password:</label>
                <input v-model="form.password" id="password" type="password" required />
            </div>
            <button type="submit" class="login-button">Login</button>
        </form>
        <button @click="goToRegister" class="register-button">Register</button>
        <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p> <!-- Display error messages -->
    </div>
</template>

<script>
import axios from 'axios';
import { mapActions } from 'vuex';

// const apiBaseUrl = process.env.VUE_APP_API_BASE_URL;

export default {
    name: 'UserLogin',
    data() {
        return {
            form: {
                email: '',
                password: '',
            },
            errorMessage: '', // Add error message data property
            apiBaseUrl: process.env.VUE_APP_API_BASE_URL,
        };
    },
    methods: {
        async login() {
            try {
                console.log('Attempting to login with:', this.form);
                const response = await axios.post(`${this.apiBaseUrl}/login`, this.form);
                console.log('Login response:', response.data);

                const { token } = response.data;
                console.log('Token received:', token);

                // Assuming userData is fetched separately if not included in the response
                const userResponse = await axios.get(`${this.apiBaseUrl}/profile`, {
                    headers: { Authorization: `Bearer ${token}` },
                });
                console.log('User profile response:', userResponse.data);
                const userData = userResponse.data;

                localStorage.setItem('user', JSON.stringify(userData)); // Save user data in localStorage
                localStorage.setItem('token', token);

                // Use Vuex actions to update authentication state and user data
                this.setUser(userData); // Dispatch setUser action
                this.setAuthenticated(true); // Dispatch setAuthenticated action

                this.$router.push('/'); // Redirect to home or another protected route
            } catch (error) {
                console.error('Error logging in:', error);
                if (error.response) {
                    console.error('Error response:', error.response.data);
                }
                this.errorMessage = 'Login failed. Please check your email and password.'; // Set error message
            }
        },
        goToRegister() {
            this.$router.push('/register');
        },
        ...mapActions(['setUser', 'setAuthenticated']),
    },
};
</script>

<style scoped>
.error-message {
    color: red;
}
</style>
