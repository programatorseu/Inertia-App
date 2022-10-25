<template>
<h1>hello World, {{name}}</h1>
<ul>
    <li v-for="framework of frameworks" v-text="framework"></li>
</ul>
</template>
<script>
export default {
    props: {
        name: String,
        frameworks: Array
    },
};
</script>