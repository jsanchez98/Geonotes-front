<template>
    <li id="blogpost"> 
      <button @click="$emit('flyTo', note.coordinates)">
        {{ text }} 
        {{id}}
        </button>
        <button id="buttonid" class=" bg-red-400 hover:bg-red-700 text-white font-bold py-1 px-1 rounded" @click="$emit('delete', index, id)">
        </button>
        <button @click="$emit('close')" v-if="!showAll">
          close
        </button>
    </li>
</template>

<script>
export default {
    props: {
        text: String,
        id: Number,
        index: Number,
        note: Object,
        showAll: Boolean
    },
    updated(){
    }
}
</script>

<style scoped>
    #blogpost {
        display: flex;
        justify-content: space-between;
        margin-bottom: 5px;
        margin-left: 20px;
        transition: 1s;
        z-index: 10;
        background-color: #ede7da;
        border-radius: 10px;
        border-style:solid;
        border-color: #aba69c;
        border-width: 1px;
    }

    #buttonid {
        margin-right: 10px;
        height: 30px;
    }
</style>