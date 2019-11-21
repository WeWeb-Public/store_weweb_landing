<template>
    <div class="loader" :class="{'loading': loading}">
        <div class="spinner">
            <div class="arc arc-1"></div>
            <div class="arc arc-2"></div>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        loading: [String, Boolean]
    },
}
</script>

<style lang="scss" scoped>
@import "../../assets/manager-css/main";

.loader {
    position: absolute;
    display: none;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(255, 255, 255, 0.5);
    align-items: center;
    justify-content: center;
    z-index: 1;

    &.loading {
        display: flex !important;
    }

    .spinner {
        display: inline-block;
        position: relative;
        animation: spinner 1.2s linear infinite;
        width: 90px;
        height: 90px;

        .arc {
            width: calc(100% - 14px);
            height: calc(100% - 14px);
            position: absolute;
            top: 0;
            left: 0;
            display: block;
            border-radius: 50%;
            border: 7px solid $ww-blue;
            animation: arc-colors 6s cubic-bezier(0.05, 0.865, 0.065, 0.99)
                infinite;

            &.arc-2 {
                transform: rotate(180deg);
                animation: arc-colors-2 6s
                    cubic-bezier(0.05, 0.865, 0.065, 0.99) infinite;
            }
        }
    }
}

@keyframes spinner {
    0% {
        transform: rotate(0deg);
    }
    100% {
        transform: rotate(360deg);
    }
}

@keyframes arc-colors {
    0% {
        border-color: $ww-blue transparent transparent transparent;
    }
    50% {
        border-color: $ww-orange transparent transparent transparent;
    }
    100% {
        border-color: $ww-blue transparent transparent transparent;
    }
}

@keyframes arc-colors-2 {
    0% {
        border-color: $ww-red transparent transparent transparent;
    }
    50% {
        border-color: $ww-green transparent transparent transparent;
    }
    100% {
        border-color: $ww-red transparent transparent transparent;
    }
}
</style>
