<template>
    <div class="bubble">
        <div class="dot"></div>
        <div class="dot"></div>
        <div class="dot"></div>
    </div>
</template>

<script>
export default {

}
</script>

<style scoped>
.bubble {
    display: flex;
    justify-content: flex-start;
    align-items: center;
    margin-top: 20px;
}

.dot {
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background-color: #bbb;
    margin-right: 5px;
    animation-name: typing;
    animation-duration: 1s;
    animation-iteration-count: infinite;
    animation-timing-function: linear;
}

.dot:nth-child(2) {
    animation-delay: 0.1s;
}

.dot:nth-child(3) {
    animation-delay: 0.2s;
}

@keyframes typing {
    0% {
        opacity: 1;
        transform: translateY(0);
    }

    50% {
        opacity: 0.5;
        transform: translateY(-10px);
    }

    100% {
        opacity: 1;
        transform: translateY(0);
    }
}
</style>