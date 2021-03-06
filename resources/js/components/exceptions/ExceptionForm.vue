<template>
    <form @submit.prevent="handleSubmit" class="form-horizontal">
        <div
            v-if="server_message !== false"
            class="alert alert-danger"
            role="alert"
        >
            {{ this.server_message }}
            <a v-if="try_logging_in" href="/login">Login</a>
        </div>

        <div class="row">
            <div class="col-md-12">
                <std-form-group
                    label="Section"
                    label-for="section"
                    :errors="form_errors.section"
                >
                    <fld-input name="section" v-model="form_data.section" />
                </std-form-group>
            </div>
        </div>

        <div class="row">
            <div class="col-md-12">
                <std-form-group
                    label="Name"
                    label-for="name"
                    :errors="form_errors.name"
                    :required="true"
                >
                    <fld-input name="name" v-model="form_data.name" required />
                    <template slot="help">
                        Name must be unique.
                    </template>
                </std-form-group>
            </div>
        </div>

        <div class="row">
            <div class="col-md-12">
                <std-form-group
                    label="Short Name"
                    label-for="short_name"
                    :errors="form_errors.short_name"
                >
                    <fld-input
                        name="short_name"
                        v-model="form_data.short_name"
                    />
                </std-form-group>
            </div>
        </div>

        <div class="row">
            <div class="col-md-12">
                <std-form-group
                    label="Attorney Note"
                    label-for="attorney_note"
                    :errors="form_errors.attorney_note"
                >
                    <fld-input
                        name="attorney_note"
                        v-model="form_data.attorney_note"
                    />
                </std-form-group>
            </div>
        </div>

        <div class="row">
            <div class="col-md-12">
                <std-form-group
                    label="Dyi Note"
                    label-for="dyi_note"
                    :errors="form_errors.dyi_note"
                >
                    <fld-input name="dyi_note" v-model="form_data.dyi_note" />
                </std-form-group>
            </div>
        </div>

        <div class="row">
            <div class="col-md-12">
                <std-form-group
                    label="Logic"
                    label-for="logic"
                    :errors="form_errors.logic"
                >
                    <fld-input name="logic" v-model="form_data.logic" />
                </std-form-group>
            </div>
        </div>

        <div class="row">
            <div class="col-md-12">
                <std-form-group
                    label="Sequence"
                    label-for="sequence"
                    :errors="form_errors.sequence"
                >
                    <fld-input name="sequence" v-model="form_data.sequence" />
                </std-form-group>
            </div>
        </div>

        <div class="form-group mt-4">
            <div class="row">
                <div class="col-md-6">
                    <button
                        type="submit"
                        class="btn btn-primary"
                        :disabled="processing"
                    >
                        <span v-if="this.form_data.id">Change Exceptions</span>
                        <span v-else="this.form_data.id">Add Exceptions</span>
                    </button>
                </div>
                <div class="col-md-6 text-md-right mt-2 mt-md-0">
                    <a href="/exception" class="btn btn-default">Cancel</a>
                </div>
            </div>
        </div>
    </form>
</template>

<script>
import axios from "axios";

export default {
    name: "exception-form",
    props: {
        record: {
            type: [Boolean, Object],
            default: false
        },
        csrf_token: {
            type: String,
            default: ""
        }
    },
    data() {
        return {
            form_data: {
                // _method: 'patch',
                _token: this.csrf_token,
                id: 0,
                section: "",
                name: "",
                short_name: "",
                attorney_note: "",
                dyi_note: "",
                logic: "",
                sequence: 0,
                deleted_at: ""
            },
            form_errors: {
                id: false,
                section: false,
                name: false,
                short_name: false,
                attorney_note: false,
                dyi_note: false,
                logic: false,
                sequence: false,
                deleted_at: false
            },
            server_message: false,
            try_logging_in: false,
            processing: false
        };
    },
    mounted() {
        if (this.record !== false) {
            // this.form_data._method = 'patch';
            Object.keys(this.record).forEach(i =>
                this.$set(this.form_data, i, this.record[i])
            );
        } else {
            // this.form_data._method = 'post';
        }
    },
    methods: {
        async handleSubmit() {
            this.server_message = false;
            this.processing = true;
            let url = "";
            let amethod = "";
            if (this.form_data.id) {
                url = "/exception/" + this.form_data.id;
                amethod = "put";
            } else {
                url = "/exception";
                amethod = "post";
            }
            await axios({
                method: amethod,
                url: url,
                data: this.form_data
            })
                .then(res => {
                    if (res.status === 200) {
                        window.location = "/exception";
                    } else {
                        this.server_message = res.status;
                    }
                })
                .catch(error => {
                    if (error.response) {
                        if (error.response.status === 422) {
                            // Clear errors out
                            Object.keys(this.form_errors).forEach(i =>
                                this.$set(this.form_errors, i, false)
                            );
                            // Set current errors
                            Object.keys(error.response.data.errors).forEach(i =>
                                this.$set(
                                    this.form_errors,
                                    i,
                                    error.response.data.errors[i]
                                )
                            );
                        } else if (error.response.status === 404) {
                            // Record not found
                            this.server_message = "Record not found";
                            window.location = "/exception";
                        } else if (error.response.status === 419) {
                            // Unknown status
                            this.server_message =
                                "Unknown Status, please try to ";
                            this.try_logging_in = true;
                        } else if (error.response.status === 500) {
                            // Unknown status
                            this.server_message =
                                "Server Error, please try to ";
                            this.try_logging_in = true;
                        } else {
                            this.server_message = error.response.data.message;
                        }
                    } else {
                        console.log(error.response);
                        this.server_message = error;
                    }
                    this.processing = false;
                });
        }
    }
};
</script>
