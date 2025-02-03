<script setup>
import { ref, computed } from 'vue';
import { useStore } from 'dashboard/composables/store';
import { useRoute } from 'vue-router';
import { useI18n } from 'vue-i18n';
import { useAlert } from 'dashboard/composables';

import Dialog from 'dashboard/components-next/dialog/Dialog.vue';

const props = defineProps({
  selectedContact: {
    type: Object,
    default: null,
  },
});

const emit = defineEmits(['goToContactsList']);

const { t } = useI18n();
const store = useStore();
const route = useRoute();

const dialogRef = ref(null);
const errorDialogRef = ref(null);

// Adicione esta computed para verificar o papel do usuário
const currentRole = computed(() => store.getters.getCurrentRole);
const isAdmin = computed(() => currentRole.value === 'administrator');

const deleteContact = async id => {
  if (!id) return;

  try {
    await store.dispatch('contacts/delete', id);
    useAlert(t('CONTACTS_LAYOUT.DETAILS.DELETE_DIALOG.API.SUCCESS_MESSAGE'));
  } catch (error) {
    useAlert(t('CONTACTS_LAYOUT.DETAILS.DELETE_DIALOG.API.ERROR_MESSAGE'));
  }
};

const handleDialogConfirm = async () => {
  if (!isAdmin.value) {
    dialogRef.value?.close();
    errorDialogRef.value?.open();
    return;
  }

  emit('goToContactsList');
  await deleteContact(route.params.contactId || props.selectedContact.id);
  dialogRef.value?.close();
};

defineExpose({ dialogRef });
</script>

<template>
  <Dialog
    ref="dialogRef"
    type="alert"
    :title="t('CONTACTS_LAYOUT.DETAILS.DELETE_DIALOG.TITLE')"
    :description="
      t('CONTACTS_LAYOUT.DETAILS.DELETE_DIALOG.DESCRIPTION', {
        contactName: props.selectedContact.name,
      })
    "
    :confirm-button-label="t('CONTACTS_LAYOUT.DETAILS.DELETE_DIALOG.CONFIRM')"
    @confirm="handleDialogConfirm"
  >
    <div 
      class="warning-message"
      :style="{
        backgroundColor: '#665417',
        padding: '12px',
        borderRadius: '4px',
        border: '1px solid rgba(0, 0, 0, 0.1)',
        marginTop: '8px',
        color: 'white'
      }"
    >
    Antes de excluir o contato, resolva todas as conversas abertas relacionadas a ele!
    </div>
  </Dialog>

  <!-- Novo diálogo para erro de permissão -->
  <Dialog
    ref="errorDialogRef"
    type="alert"
    title="Permissão necessária"
    description="Somente administradores podem excluir contatos. Não foi possível excluir o contato."
    confirm-button-label="OK"
  />
</template>