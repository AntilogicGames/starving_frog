<template>
    <div id="app">
        <Frog/>
    </div>
</template>

<script>
    import Frog from './components/Frog'

    export default {
        name: 'App',
        components: {
            Frog
        }
    }
</script>

<style>
    body {
        background-image: url('./assets/background.jpg');
        background-repeat: no-repeat;
        background-size: cover;
        background-color: black;
        color: white;
        font-family: monospace;
    }
</style>