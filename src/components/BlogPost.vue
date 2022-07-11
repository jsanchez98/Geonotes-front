<template>
    <li id="blogpost"> 
        {{ text }} 
        <button id="buttonid" class=" bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" @click="$emit('delete', index, id)">
            DEL
        </button>
    </li>
</template>

<script>
export default {
    props: {
        text: String,
        id: Number,
        index: Number
    },
    methods: {
    },
}
</script>

<style scoped>
    #blogpost {
        display: flex;
        justify-content: space-between;
        margin-bottom: 5px;
        margin-left: 20px;
    }

    #buttonid {
        margin-right: 20px;
    }
</style>