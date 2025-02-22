<template>
    <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
      <div class="bg-white dark:bg-slate-800 rounded-lg p-6 w-full max-w-md">
        <h3 class="text-lg font-medium mb-4 text-slate-800 dark:text-slate-200">Nova Coluna</h3>
        <input
          v-model="title"
          type="text"
          placeholder="Nome da coluna"
          class="w-full px-3 py-2 border rounded-md mb-4 dark:bg-slate-700 dark:border-slate-600"
        />
        <div class="flex justify-end space-x-2">
          <button @click="$emit('close')" class="btn-secondary">Cancelar</button>
          <button @click="submit" class="btn-primary">Criar</button>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return { title: '' };
    },
    methods: {
      submit() {
        if (this.title.trim()) {
          this.$emit('submit', this.title);
          this.$emit('close');
        }
      }
    }
  };
  </script>