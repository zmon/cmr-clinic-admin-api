/**

For select specifically for please_expunges
Example usage:
=====================

<fld-group-no
    name="please_expunge"
    v-model="form_data.please_expunge"
    required
/>

*/

<template>
    <select
        class="form-control"
        :value="modelValue"
        @change="updateValue"
        :name="this.name"
        :id="'field_' + this.name">
        <option v-for="c in please_expunges" v-bind:value="c.value"> {{ c.text }}</option>
    </select>

</template>

<script>
    export default {
        name: "fld-group-no",
        model: {
            prop: 'modelValue',
            event: 'change'
        },
        props: {
            modelValue: {
                type: [Number, String],
                default: ''
            },
            name: {
                type: String,
                default: ''
            }
        },
        data() {
            return {
                selected: '',
                please_expunges: [
                    {
                        value: 0,
                        text: '-- Select --'
                    },
                    {
                        value: 1,
                        text: '1'
                    },
                    {
                        value: 2,
                        text: '2'
                    }, {
                        value: 3,
                        text: '3'
                    },
                ]
            }
        },

        mounted: function () {
            this.selected = this.modelValue;
        },

        methods: {
            updateValue: function (event) {
                this.$emit('change', event.target.value);
            },

        }


    }
</script>


