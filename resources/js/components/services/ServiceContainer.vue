<template>
    <div>
        <h5>
            Service List
            <span @click="addService" class="add-button">+</span>
        </h5>
        <table class="table  table-sm">
            <tr  v-for="(service,i) in services"
                :key="service.id + 'service-' + i"
            >
                <td class="service-name-column">
                   <span class="service-name-field">
                        {{service.service_type ? service.service_type.name : ''}}
                    </span>
                </td>
                <td>
                    {{service.name}}
                </td>
                <td class="d-inline-block">
                    <pencil-control height="25"
                                    class="d-inline-flex"
                                    @click="editService(service, i)"/>
                    <delete-control
                        class="d-inline-flex"
                        height="25"
                        @click="deleteService(service, i)"
                    ></delete-control>
                </td>
            </tr>
        </table>
        <base-modal v-if="showServiceModal" @close="closeModal">
            <template v-slot:header>Case Service</template>
            <template v-slot:body>
                <std-form-group :errors="form_errors.service.name">
                    <label class="font-weight-bold">Service Name</label>
                    <autocomplete
                            style="width: 100%"
                            ref="autocomplete"
                            url="/services/all"
                            :create="true"
                            :value="selectedService.name"
                            valueField="name"
                            displayField="address"
                            @selected="selectService"
                            @input="updateService"
                            @create="addNew"
                            :disabled="disableService"
                    ></autocomplete>
                </std-form-group>
                <std-form-group :errors="form_errors.service.service_type_id">
                    <label class="font-weight-bold">Service Type</label>
                    <select class="form-control" v-model="selectedService.service_type_id" >
                        <option value="">--Select--</option>
                        <option v-for="type in serviceTypes" :value="type.id">{{type.name}}</option>
                    </select>
                </std-form-group>


                <std-form-group :errors="form_errors.service.address">
                    <label class="font-weight-bold">Address</label>
                    <input type="text" required class="form-control" placeholder="Address"
                           v-model="selectedService.address"
                           :disabled="disableFields"
                    >
                </std-form-group>

                <div class="row">
                <div class="col-md-6">
                    <std-form-group label="City" label-for="city" :errors="form_errors.city">
                        <fld-input
                            name="city"
                            v-model="selectedService.city"
                            :disabled="disableFields"
                        />
                    </std-form-group>
                </div>
                <div class="col-md-2">
                    <std-form-group label="State" label-for="state" :errors="form_errors.state">
                        <fld-state
                            name="state"
                            v-model="selectedService.state"
                            :disabled="disableFields"
                        />
                    </std-form-group>
                </div>
                <div class="col-md-4">
                    <std-form-group label="Zip" label-for="zip_code" :errors="form_errors.zip">
                        <fld-input
                            name="zip_code"
                            v-model="selectedService.zip_code"
                            :disabled="disableFields"
                        />
                    </std-form-group>
                </div>
                </div>
                <std-form-group :errors="form_errors.service.note">
                    <label class="font-weight-bold">Note</label>
                    <textarea type="text" required class="form-control"
                              placeholder="Note"
                              v-model="selectedService.note"
                              :disabled="disableFields"
                    ></textarea>
                </std-form-group>

            </template>
            <template v-slot:footer>
                <button class="btn btn-secondary float-left" @click.prevent="closeModal">Cancel</button>
                <div v-if="disableService">
                    <button type="submit" class="btn btn-danger float-right" @click.prevent="submitDelete">Delete
                    </button>
                    <button type="submit" class="btn btn-primary float-right" @click.prevent="submitEdit">Edit</button>
                </div>
                <div v-else>
                    <button type="submit" class="btn btn-primary float-right" @click.prevent="submitCreate">Create
                    </button>
                </div>
            </template>
        </base-modal>
    </div>
</template>

<script>
    import PencilControl from "../controls/PencilControl";

    import DeleteControl from "../controls/DeleteControl";
    export default {
        components: {PencilControl, DeleteControl},
        name: "ServiceContainer",
        props: {
            services: {
                type: Array | Object,

            },
            case_id: {
                type: Number,
            }
        },
        data() {
            return {
                showServiceModal: false,
                selectedService: {
                    name: ''
                },
                disableFields: true,
                disableService: false,
                newService: {
                    id: null,
                    name: '',
                    address: null,
                    address_line_2: null,
                    city: null,
                    state: null,
                    zip: null,
                    phone: null,
                    email: null,
                    note: null,
                    service_type_id: null,
                    pivot: {
                        name: ''
                    }
                },
                serviceTypes: [],
                selectedIndex: null,
                form_errors: {
                    name: false,
                    service: {
                        id: false,
                        name: false,
                        address: false,
                        phone: false,
                        email: false,
                        note: false,
                        service_type_id: false,
                    }
                }
            }
        },
        mounted() {
            this.getServiceTypes()
        },
        methods: {
            addNew() {
                this.disableFields = false
                let name = this.selectedService.name
                Object.assign(this.selectedService, this.newService)
                this.selectedService.name = name
            },
            addService() {
                this.selectedIndex = null
                Object.assign(this.selectedService, this.newService)
                this.showServiceModal = true
                this.disableService = false

            },
            selectService(v) {
                console.log('selectService', v, this.selectedService)

                if (!this.selectedService.attn) {
                    this.selectedService.attn = "Court Clerk";
                }

                this.selectedService.pivot.name = this.selectedService.attn;

                // keep the name of the pivot so it is overwritten
                v.pivot = {name: this.selectedService.pivot.name}
                Vue.set(this, 'selectedService', v)

                this.disableFields = true
            },
            editService(s, i) {
                this.selectedIndex = i
                this.disableService = true
                this.disableFields = true
                Object.assign(this.selectedService, s)
                this.showServiceModal = true
            },
            submitCreate() {
                console.log('create service')
                // if (!this.selectedService.attn) {
                //     this.selectedService.attn = "Court Clerk";
                // }
                //
                // this.selectedService.pivot.name = this.selectedService.attn;

                let $this = this
                axios.post(`/case/${this.case_id}/service/create`, {
                    name: this.selectedService.pivot.name,
                    service: this.selectedService,
                }).then(res => {
                    console.log(res)
                    $this.selectedService.id = res.data.id
                    $this.selectedService.service_type = res.data.service_type
                    $this.$emit('created', this.selectedService)
                    this.closeModal()
                }).catch(e => {
                    for (let name in e.response.data.errors) {
                        let split = name.split('.')
                        if (split.length > 1) {
                            $this.form_errors[split[0]][split[1]] = e.response.data.errors[name]
                        } else {
                            $this.form_errors[name] = e.response.data.errors[name]
                        }
                    }

                })
            },
            deleteService(s, i) {
                this.selectedIndex = i
                Object.assign(this.selectedService, s)
                this.submitDelete()
            },
            submitDelete() {
                console.log('delete service')
                let $this = this
                axios.delete(`/case/${this.case_id}/service/${this.selectedService.id}`, {
                    name: this.selectedService.pivot.name,
                    service: this.selectedService,
                }).then(res => {
                    console.log(res)
                    this.$emit('deleted', this.selectedService, this.selectedIndex)
                    this.closeModal()
                }).catch(e => {
                    console.error(e)
                })
            },
            submitEdit() {
                console.log('edit service')
                let $this = this
                axios.put(`/case/${this.case_id}/service/${this.selectedService.id}`, {
                    name: this.selectedService.pivot.name,
                    service: this.selectedService,
                }).then(res => {
                    console.log(res)
                    this.$emit('updated', this.selectedService, this.selectedIndex)
                    this.closeModal()
                }).catch(e => {
                    console.error(e)
                    for (let name in e.response.data.errors) {
                        $this.form_errors[name] = e.response.data.errors[name]
                    }
                })
            },
            // called when input changes
            updateService(v) {
                console.log('updateService', v)
                this.selectedService.name = v
            },
            getServiceTypes() {
                axios.get('/service-types/all')
                    .then(res => {
                        this.serviceTypes = res.data
                    })
                    .catch(e => {
                        console.log(e)
                    })
            },
            resetErrors() {
                this.form_errors = {
                    name: false,
                    service: {
                        id: false,
                        name: false,
                        address: false,
                        phone: false,
                        email: false,
                        note: false,
                        service_type_id: false,
                    }
                }
            },
            closeModal() {
                this.showServiceModal = false
                this.resetErrors()
            }
        }
    }
</script>

<style scoped>
    .service-name-column {
        width: 12em;
    }

    .service-name-field {
        color: darkgray;
    }

    .name-field:hover {
        font-weight: bolder;
        color: #343a40;
        cursor: pointer;
    }

    .add-button {
        color: #aaaaaa;
        font-weight: normal;
    }

    .add-button:hover {
        color: #343a40;
        font-weight: bolder;
        cursor: pointer;
    }

</style>
