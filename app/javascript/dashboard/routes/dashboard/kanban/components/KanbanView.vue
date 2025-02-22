<template>
  <div class="kanban-container h-full bg-white dark:bg-slate-900 flex flex-col">
    <!-- Header do Kanban -->
    <div class="p-4 border-b border-slate-200 dark:border-slate-700 flex items-center justify-between">
      <div class="flex items-center space-x-4">
        <h1 class="text-xl font-bold text-slate-800 dark:text-slate-200">{{ boardName }}</h1>
        <span class="beta-tag">BETA</span>
        <button
          @click="showColumnModal = true"
          class="flex items-center space-x-1 text-woot-500 hover:text-woot-600 dark:hover:text-woot-400 transition-colors"
        >
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"/>
          </svg>
          <span class="text-sm font-medium">Adicionar Coluna</span>
        </button>
      </div>
    </div>

    <!-- Área do Kanban -->
    <div class="flex-1 overflow-x-auto p-4">
      <div class="flex space-x-4">
        <draggable 
          v-model="columns"
          group="columns"
          item-key="id"
          class="flex space-x-4"
          ghost-class="opacity-50"
          handle=".column-handle"
          @end="saveState"
        >
          <template #item="{ element: column }">
            <div class="kanban-column flex-shrink-0 w-72 bg-slate-50 dark:bg-slate-800 rounded-lg shadow-sm border border-slate-200 dark:border-slate-700">
              <!-- Header da Coluna -->
              <div class="p-4 border-b border-slate-200 dark:border-slate-700 flex items-center justify-between">
                <div class="flex items-center space-x-2 column-handle cursor-move">
                  <span class="text-slate-400 dark:text-slate-500">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"/>
                    </svg>
                  </span>
                  <h3 class="font-medium text-sm text-slate-700 dark:text-slate-200">
                    {{ column.title }}
                  </h3>
                </div>
                <div class="flex items-center space-x-2">
                  <span class="px-2 py-1 text-xs font-medium rounded-full bg-slate-200 dark:bg-slate-700 text-slate-600 dark:text-slate-300">
                    {{ columnTickets(column.id).length }}
                  </span>
                  <button 
                    @click="openTaskModal(column.id)"
                    class="text-slate-400 hover:text-woot-500 dark:hover:text-woot-400 transition-colors"
                  >
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"/>
                    </svg>
                  </button>
                </div>
              </div>

              <!-- Tickets -->
              <draggable
                :modelValue="columnTickets(column.id)"
                @update:modelValue="handleTicketUpdate($event, column.id)"
                group="tickets"
                item-key="id"
                class="p-2 min-h-[150px]"
                :class="{ 'shift-dragging': isShiftPressed }"
                :ghost-class="isShiftPressed ? 'shift-ghost' : 'opacity-50'"
                animation="200"
                @end="saveState"
              >
                <template #item="{ element: ticket }">
                  <div class="ticket-card group mb-2 p-3 bg-white dark:bg-slate-700 rounded-md shadow-xs border border-slate-200 dark:border-slate-600 cursor-move hover:border-woot-500 dark:hover:border-woot-500 transition-all duration-200">
                    <div class="flex items-start justify-between">
                      <div class="flex-1">
                        <div class="text-sm font-medium text-slate-700 dark:text-slate-100 mb-1">
                          {{ ticket.title }}
                        </div>
                        <p class="text-xs text-slate-500 dark:text-slate-400 line-clamp-2">
                          {{ ticket.content }}
                        </p>
                      </div>
                      <button 
                        @click.stop="openEditMenu(ticket, $event)"
                        class="opacity-0 group-hover:opacity-100 text-slate-400 hover:text-woot-500 transition-opacity"
                      >
                        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 5v.01M12 12v.01M12 19v.01M12 6a1 1 0 110-2 1 1 0 010 2zm0 7a1 1 0 110-2 1 1 0 010 2zm0 7a1 1 0 110-2 1 1 0 010 2z"/>
                        </svg>
                      </button>
                    </div>
                    <div class="mt-2 flex items-center justify-between text-xs">
                      <span class="text-woot-500 dark:text-woot-400">#{{ ticket.id }}</span>
                      <span class="text-slate-400">{{ formatDate(ticket.createdAt) }}</span>
                    </div>
                  </div>
                </template>
              </draggable>
            </div>
          </template>
        </draggable>
      </div>
    </div>

    <!-- Modals -->
    <div v-if="showColumnModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
      <div class="bg-white dark:bg-slate-800 rounded-lg p-6 w-full max-w-md">
        <h3 class="text-lg font-medium mb-4 text-slate-800 dark:text-slate-200">Nova Coluna</h3>
        <input
          v-model="newColumnTitle"
          type="text"
          placeholder="Nome da coluna"
          class="w-full px-3 py-2 border rounded-md mb-4 dark:bg-slate-700 dark:border-slate-600"
          @keyup.enter="addColumn"
        />
        <div class="flex justify-end space-x-2">
          <button @click="showColumnModal = false" class="btn-secondary">Cancelar</button>
          <button @click="addColumn" class="btn-primary">Criar</button>
        </div>
      </div>
    </div>

    <div v-if="showTaskModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
      <div class="bg-white dark:bg-slate-800 rounded-lg p-6 w-full max-w-md">
        <h3 class="text-lg font-medium mb-4 text-slate-800 dark:text-slate-200">Nova Tarefa</h3>
        <input
          v-model="newTaskTitle"
          type="text"
          placeholder="Título"
          class="w-full px-3 py-2 border rounded-md mb-4 dark:bg-slate-700 dark:border-slate-600"
          @keyup.enter="addTask"
        />
        <textarea
          v-model="newTaskContent"
          placeholder="Descrição"
          class="w-full px-3 py-2 border rounded-md mb-4 h-24 dark:bg-slate-700 dark:border-slate-600"
        ></textarea>
        <div class="flex justify-end space-x-2">
          <button @click="showTaskModal = false" class="btn-secondary">Cancelar</button>
          <button @click="addTask" class="btn-primary">Criar</button>
        </div>
      </div>
    </div>

    <div 
      v-if="selectedTicket" 
      class="fixed inset-0 z-40" 
      @click.self="selectedTicket = null"
      :style="{ top: contextMenuPosition.y + 'px', left: contextMenuPosition.x + 'px' }"
    >
      <div class="absolute bg-white dark:bg-slate-700 rounded-md shadow-lg p-2 min-w-[160px]">
        <button 
          @click="deleteTicket"
          class="w-full text-left px-3 py-2 text-sm hover:bg-slate-100 dark:hover:bg-slate-600 rounded-md text-red-500"
        >
          Excluir
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import { defineComponent } from 'vue';
import draggable from 'vuedraggable';

export default defineComponent({
  components: { draggable },
  props: {
    boardName: {
      type: String,
      default: 'Kanban'
    }
  },
  data() {
    return {
      columns: JSON.parse(localStorage.getItem('kanbanColumns')) || [
        { id: 'open', title: 'Aberto' },
        { id: 'in_progress', title: 'Em Progresso' },
        { id: 'done', title: 'Concluído' }
      ],
      tickets: JSON.parse(localStorage.getItem('kanbanTickets')) || [],
      showColumnModal: false,
      showTaskModal: false,
      newColumnTitle: '',
      newTaskTitle: '',
      newTaskContent: '',
      selectedColumn: null,
      selectedTicket: null,
      isShiftPressed: false,
      contextMenuPosition: { x: 0, y: 0 }
    };
  },
  methods: {
    columnTickets(status) {
      return this.tickets.filter(ticket => ticket.status === status);
    },
    handleTicketUpdate(newList, columnId) {
      newList.forEach(ticket => {
        if (ticket.status !== columnId) {
          ticket.status = columnId;
        }
      });
      this.saveState();
    },
    addColumn() {
      if (this.newColumnTitle.trim()) {
        this.columns.push({
          id: `col-${Date.now()}`,
          title: this.newColumnTitle
        });
        this.newColumnTitle = '';
        this.showColumnModal = false;
        this.saveState();
      }
    },
    openTaskModal(columnId) {
      this.selectedColumn = columnId;
      this.showTaskModal = true;
    },
    addTask() {
      if (this.newTaskTitle.trim()) {
        this.tickets.push({
          id: Date.now(),
          title: this.newTaskTitle,
          content: this.newTaskContent,
          status: this.selectedColumn,
          createdAt: new Date()
        });
        this.newTaskTitle = '';
        this.newTaskContent = '';
        this.showTaskModal = false;
        this.saveState();
      }
    },
    openEditMenu(ticket, event) {
      this.selectedTicket = ticket;
      this.contextMenuPosition = {
        x: event.clientX,
        y: event.clientY
      };
    },
    deleteTicket() {
      this.tickets = this.tickets.filter(t => t.id !== this.selectedTicket.id);
      this.selectedTicket = null;
      this.saveState();
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString('pt-BR', {
        day: '2-digit',
        month: 'short',
        year: 'numeric'
      });
    },
    saveState() {
      localStorage.setItem('kanbanColumns', JSON.stringify(this.columns));
      localStorage.setItem('kanbanTickets', JSON.stringify(this.tickets));
    },
    handleKeyDown(e) {
      if (e.key === 'Shift') this.isShiftPressed = true;
    },
    handleKeyUp(e) {
      if (e.key === 'Shift') this.isShiftPressed = false;
    }
  },
  mounted() {
    window.addEventListener('keydown', this.handleKeyDown);
    window.addEventListener('keyup', this.handleKeyUp);
  },
  beforeUnmount() {
    window.removeEventListener('keydown', this.handleKeyDown);
    window.removeEventListener('keyup', this.handleKeyUp);
  }
});
</script>

<style scoped>
.kanban-container {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.kanban-column {
  height: calc(100vh - 180px);
}

.ticket-card {
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.ticket-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.ticket-card.shift-dragging {
  transform: rotate(3deg) scale(1.05) !important;
  border: 2px solid #6cb2eb !important;
  box-shadow: 0 4px 12px rgba(108, 178, 235, 0.3) !important;
}

.ghost-class.shift-ghost {
  opacity: 0.8 !important;
  transform: rotate(-3deg) scale(0.95);
} 

.beta-tag {
  @apply text-[10px] font-medium px-1 py-0.5 rounded-full bg-transparent text-[#33ae73] border border-[#33ae73];
  transform: translateY(-1px);
  letter-spacing: 0.5px;
}

.dark .ticket-card {
  background: #1e293b;
  border-color: #334155;
}

.dark .kanban-column {
  background: #0f172a;
  border-color: #1e293b;
}

.btn-primary {
  @apply bg-woot-500 text-white px-4 py-2 rounded-md hover:bg-woot-600 transition-colors;
}

.btn-secondary {
  @apply bg-slate-200 dark:bg-slate-700 text-slate-800 dark:text-slate-200 px-4 py-2 rounded-md hover:bg-slate-300 dark:hover:bg-slate-600 transition-colors;
}
</style>