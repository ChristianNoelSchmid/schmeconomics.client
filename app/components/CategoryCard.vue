<script setup lang="ts">
import { currencyFormat } from '~/formatters';
import { Role, type CategoryModel } from '~/lib/openapi';
import { useSignInState } from '~/lib/services/auth-state';

const signInState = useSignInState();
const props = defineProps<{
  category: CategoryModel
}>();
const balance = computed(() => currencyFormat(props.category.balance));

const emit = defineEmits<{
  deleteclicked: []
  editclicked: []
}>();
</script>

<template>
  <UCard class="relative">
    <template #header>
      <div class="flex justify-between items-center">
        <h3 class="text-lg font-semibold">{{ props.category.name }}</h3>
      </div>
    </template>

    <p>Balance: {{ balance }}</p>

    <template #footer>
      <div class="flex justify-end space-x-2">
        <UButton color="success" variant="ghost" icon="i-heroicons-plus-circle" size="xl" />
        <UButton color="error" variant="ghost" icon="i-heroicons-minus-circle" size="xl" />
        <UButton v-if="signInState?.userModel.role == Role.Admin" color="warning" variant="ghost"
          icon="i-heroicons-pencil" size="xl" @click="emit('editclicked')" />
        <UButton v-if="signInState?.userModel.role == Role.Admin" color="error" variant="ghost" icon="i-heroicons-trash"
          size="xl" @click="emit('deleteclicked')" />
      </div>
    </template>
  </UCard>
</template>
