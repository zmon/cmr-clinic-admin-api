<template>
    <div>
        <div class="row">
            <div class="col-md-12">
                <div class=" charge-container">


                    <applicant-details :can-cms="this.canCms" v-model="record">

                    </applicant-details>

                    <div class="row">
                        <div class="col-md-12">
                            <!--For now we will keep this one passed as a propery-->
                            <cases-show-list :data="this.record.conviction"
                                             :client_id="this.record.id"></cases-show-list>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import ApplicantDetails from "./ApplicantDetails";
    import CasesShowList from "../cases/CasesShowList";

    export default {
        name: "applicant-show",
        components: {
            ApplicantDetails,
            CasesShowList
        },
        props: {
            data: {
                type: [Boolean, Object, Array],
                default: () => {
                    return {
                        id: 0,
                        name: null,
                        step_id: null
                    }
                },
            },
            canCms: {
                type: [Boolean, Number],
                default: false
            }
        },
        data() {
            return {
                record: {}
            }
        },
        created() {
            Object.keys(this.data).forEach(i =>
                this.$set(this.record, i, this.data[i])
            );
        },
    }
</script>

