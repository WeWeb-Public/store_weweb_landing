<template>
    <div class="ww-search-bar-wrapper">
        <div class="fas fa-search fa-flip-horizontal"></div>
        <input class="search-input" type="text" :placeholder="placeholder" v-model="internalValue" />
    </div>
</template>

<script> 
import Vue from "vue";

export default {
    name: "wwManagerSearchBarIcon",
    props: {
        placeholder: String,
        search: String,
        value: String,
        debounce: [String, Number],

    },
    data() {
        return {
            internalValue: '',
            timeout: null
        };
    },
    watch: {
        internalValue() {
            let self = this;
            clearTimeout(this.timeout);

            this.timeout = setTimeout(function () {
                self.$emit('input', self.internalValue);
                self.$emit('change', self.internalValue);
            }, this.debounceMs);
        }
    },
    computed: {
        debounceMs() {
            try {
                return parseInt(this.debounce);
            } catch (error) {
                return 0;
            }
        }
    },
    methods: {
    },
    mounted: function () {
        this.internalValue = this.value;
    },
    beforeDestroy() {

    }
};
</script>

<style scoped lang="scss">
@import "../assets/manager-css/main";
.ww-search-bar-wrapper {
    border: 1px solid $ww-grey-strong;
    display: flex;
    justify-content: space-evenly;
    align-items: center;
    flex-direction: row;
    font-family: $ww-font;
    border-radius: 15px;
    padding: 3px 10px;
    color: $ww-grey-super-light;
    font-size: 1rem;
    background-color: white;
    &:hover {
        border: 1px solid $ww-grey-light;
    }
    .search-input {
        font-size: 0.7rem;
        border: none;
        margin-left: 10px;
        flex-grow: 1;
        color: $ww-grey-light;
        overflow: hidden;
        text-overflow: ellipsis;
        &:focus {
            outline: none;
        }
    }

    .fas {
        font-size: 0.9rem;
        color: $ww-grey-super-light;
    }
    ::placeholder {
        /* Chrome, Firefox, Opera, Safari 10.1+ */
        color: $ww-grey-super-light;
        opacity: 1; /* Firefox */
    }

    :-ms-input-placeholder {
        /* Internet Explorer 10-11 */
        color: $ww-grey-super-light;
    }

    ::-ms-input-placeholder {
        /* Microsoft Edge */
        color: $ww-grey-super-light;
    }
}
</style>