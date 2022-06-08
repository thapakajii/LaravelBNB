<template>
    <div>
        <h6 class="text-uppercase text-secondary font-weight-bolder">
            Check Availability
            <transition>
                <span v-if="noAvailability" class="text-danger">(Not Available)</span>
                <span v-if="hasAvailability" class="text-success">(Available)</span>
            </transition>
        </h6>
        <div class="row">
            <div class="form-group col-md-6">
                <label for="from">From</label>
                <input 
                    type="text"
                    name="from"
                    class="form-control form-control-sm"
                    placeholder="Start date"
                    v-model="from"
                    @keyup.enter="check"
                    :class="[{'is-invalid': errorFor('from')}]"/>
                <v-errors :errors="errorFor('from')"></v-errors>

            </div>
            <div class="form-group col-md-6">
                <label for="to">To</label>
                <input 
                    type="text"
                    name="to"
                    class="form-control form-control-sm"
                    placeholder="End date"
                    v-model="to"
                    @keyup.enter="check"
                    :class="[{'is-invalid': errorFor('to')}]"
                />
                <v-errors :errors="errorFor('to')"></v-errors>

            </div>
        </div>
        <button class="btn btn-secondary w-100 mt-4" @click="check" :disabled="loading">
            <span v-if="!loading">Check!</span> 
            <span v-if="loading"><i class="fas fa-circle-notch fa-spin"></i>Checking...</span> 
        </button>
    </div>
</template>

<script>
import {is422} from "./../shared/utilis/response";
import validationErrors from "./../shared/mixins/validationErrors";

export default {
    mixins: [validationErrors],
    props: {
        bookableId: [String, Number]
    },
    data(){
        return{
            from: this.$store.state.lastSearch.from,
            to: this.$store.state.lastSearch.to,
            loading: false,
            status: null
        };
    },
    methods: {
        async check(){
            this.loading = true;
            this.errors = null;

            this.$store.dispatch("setLastSearch", {
                from: this.from,
                to: this.to
            });

            try {
                this.status = (await axios.get(
                    `/api/bookables/${this.bookableId}/availability?from=${this.from}&to=${this.to}`
                )).status;
                this.$emit("availability", this.hasAvailability);
            } catch (err){
                if (is422(err)){
                    this.errors = err.response.data.errors;
                }

                this.status = err.response.status;
                this.$emit("availability", this.hasAvailability);
            }

            this.loading = false;
        }
    },
    computed: {
        hasErrors(){
            return 422 === this.status && this.errors !== null;
        },
        hasAvailability(){
            return 200 === this.status;
        },
        noAvailability(){
            return 404 === this.status;
        }
    }
};
</script>

<style scoped>
label{
    font-size: 0.7rem;
    text-transform: uppercase;
    color: gray;
    font-weight: bolder;
}
</style>