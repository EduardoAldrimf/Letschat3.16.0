<template>
    <div class="fixed inset-0 z-40" @click.self="$emit('close')">
      <div class="absolute bg-white dark:bg-slate-700 rounded-md shadow-lg p-2 min-w-[160px]" 
           :style="{ left: `${position.x}px`, top: `${position.y}px` }">
        <button @click="$emit('delete')" class="w-full text-left px-3 py-2 text-sm hover:bg-slate-100 dark:hover:bg-slate-600 rounded-md text-red-500">
          Excluir
        </button>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    props: ['ticket'],
    data() {
      return {
        position: { x: 0, y: 0 }
      };
    },
    mounted() {
      this.position = { x: event.clientX, y: event.clientY };
    }
  };
  </script>