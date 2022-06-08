<template>
    <div class="w-50 m-auto">
        <div class="card card-body">
            <form>
                <div class="form-group">
                    <label for="email">E-mail</label>
                    <input 
                        type="text"
                        name="email"
                        placeholder="Enter your e-mail"
                        class="form-control mt-2"
                        v-model="email"
                        :class="[{'is-invalid': errorFor('email')}]" 
                    >
                    <v-errors :errors="errorFor('email')"></v-errors>
                </div>
                <div class="form-group">
                    <label for="password" class="mt-2">Password</label>
                    <input 
                        type="password"
                        name="password"
                        placeholder="Enter your password"
                        class="form-control mt-2"
                        v-model="password"
                        :class="[{'is-invalid': errorFor('password')}]" 
                    >
                    <v-errors :errors="errorFor('password')"></v-errors>
                </div>
                <button 
                    type="submit" 
                    class="btn btn-primary btn-lg w-100 mt-4" 
                    :disabled="loading"
                    @click.prevent="login"
                >Log In</button>
                <hr />
                <div>
                    No account yet?
                    <router-link :to="{ name: 'register' }" class="font-weight-bold text-decoration-none">Register Now</router-link>
                </div>
                <div>
                    Forgot your password?
                    <router-link :to="{ name: 'home' }" class="font-weight-bold text-decoration-none">Reset Password</router-link>
                </div>
            </form>
        </div>
    </div>
</template>

<script>
import validationErrors from '../shared/mixins/validationErrors';
import { logIn } from "./../shared/utilis/auth";

export default {
    mixins: [validationErrors],
    data(){
        return {
            email: null,
            password: null,
            loading: false
        };
    },
    methods: {
        async login(){
            this.loading = true;
            this.errors = null;

                try {
                    await axios.get("/sanctum/csrf-cookie");
                    await axios.post("/login", {
                        email: this.email,
                        password: this.password
                });

                logIn();
                this.$store.dispatch("loadUser");
                this.$router.push({ name: "home" });
                } catch (error){
                    this.errors = error.response && error.response.data.errors;
                }

            this.loading = false;
        }
    }
};
</script>